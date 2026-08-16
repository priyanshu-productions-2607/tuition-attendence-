<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Tuition Attendance</title>
<style>
*{box-sizing:border-box;font-family:Arial,sans-serif}
body{margin:0;background:#f4f6f8;color:#172033}
header{background:#2563eb;color:#fff;padding:20px;text-align:center}
header h1{margin:0 0 6px;font-size:25px}.container{max-width:650px;margin:auto;padding:14px}
.card{background:#fff;padding:16px;margin-bottom:14px;border-radius:16px;box-shadow:0 3px 12px #00000012}
input,button{width:100%;padding:12px;margin-top:8px;border-radius:9px;font-size:16px}
input{border:1px solid #cbd5e1}button{border:0;font-weight:700;cursor:pointer}
.blue{background:#2563eb;color:#fff}.green{background:#16a34a;color:#fff}
.datebox{display:flex;gap:8px;align-items:center}.datebox input{margin:0;flex:1}
.student{display:flex;align-items:center;gap:8px;padding:12px 0;border-bottom:1px solid #e5e7eb}
.name{flex:1;font-weight:700}.small{font-size:12px;color:#64748b;font-weight:400}
.actions{display:flex;gap:5px}.actions button{width:auto;margin:0;padding:9px 12px}
.p{background:#16a34a;color:#fff}.a{background:#dc2626;color:#fff}.selected{outline:3px solid #93c5fd}
.report{padding:12px;margin-top:8px;background:#f8fafc;border-radius:10px}
.history{display:flex;justify-content:space-between;align-items:center;padding:9px 0;border-bottom:1px solid #e5e7eb}
.history button{width:auto;margin:0;padding:7px 10px;background:#e2e8f0}
.empty{text-align:center;color:#64748b;padding:15px}
</style>
</head>
<body>
<header>
<h1>📚 Tuition Attendance</h1>
<div id="todayText"></div>
</header>

<div class="container">

<div class="card">
<h2>📅 Select Attendance Date</h2>
<div class="datebox">
<input type="date" id="datePicker">
<button class="blue" onclick="loadDate()">Load</button>
</div>
<p id="selectedDateText"></p>
</div>

<div class="card">
<h2>➕ Add Student</h2>
<input id="studentName" placeholder="Student name">
<input id="batchName" placeholder="Batch / Class (e.g. Class 7)">
<button class="blue" onclick="addStudent()">Add Student</button>
</div>

<div class="card">
<h2>📝 Attendance</h2>
<div id="studentList"></div>
<button class="green" onclick="saveAttendance()">💾 Save Attendance</button>
</div>

<div class="card">
<h2>📊 Student Reports</h2>
<div id="report"></div>
</div>

<div class="card">
<h2>🗓️ Saved Dates</h2>
<div id="history"></div>
</div>

</div>

<script>
let students=JSON.parse(localStorage.getItem("students")||"[]");
let attendance=JSON.parse(localStorage.getItem("attendance")||"{}");
let selectedDate=new Date().toISOString().slice(0,10);

const datePicker=document.getElementById("datePicker");
datePicker.value=selectedDate;
document.getElementById("todayText").textContent=new Date().toLocaleDateString("en-IN",{day:"numeric",month:"long",year:"numeric"});

function formatDate(d){
 return new Date(d+"T00:00:00").toLocaleDateString("en-IN",{day:"numeric",month:"long",year:"numeric"});
}
function escapeHTML(s){
 return String(s).replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;").replace(/"/g,"&quot;").replace(/'/g,"&#039;");
}
function loadDate(){
 selectedDate=datePicker.value;
 if(!selectedDate){alert("Please select a date.");return;}
 document.getElementById("selectedDateText").textContent="Selected: "+formatDate(selectedDate);
 showStudents(); showReport(); showHistory();
}
function addStudent(){
 const name=document.getElementById("studentName").value.trim();
 const batch=document.getElementById("batchName").value.trim()||"General";
 if(!name){alert("Enter student name.");return;}
 students.push({id:Date.now(),name,batch});
 localStorage.setItem("students",JSON.stringify(students));
 document.getElementById("studentName").value="";
 document.getElementById("batchName").value="";
 showStudents();showReport();
}
function showStudents(){
 const list=document.getElementById("studentList");list.innerHTML="";
 if(!students.length){list.innerHTML='<div class="empty">No students added yet.</div>';return;}
 const day=attendance[selectedDate]||{};
 students.forEach(s=>{
  const status=day[s.id]||"";
  const div=document.createElement("div");div.className="student";
  div.innerHTML=`<div class="name">${escapeHTML(s.name)}<div class="small">${escapeHTML(s.batch)}</div></div>
  <div class="actions">
  <button class="p ${status==="Present"?"selected":""}" onclick="mark(${s.id},'Present')">P</button>
  <button class="a ${status==="Absent"?"selected":""}" onclick="mark(${s.id},'Absent')">A</button>
  </div>
  <button onclick="deleteStudent(${s.id})" style="width:auto;margin:0;background:#f1f5f9;color:#dc2626">🗑️</button>`;
  list.appendChild(div);
 });
}
function mark(id,status){
 if(!attendance[selectedDate]) attendance[selectedDate]={};
 attendance[selectedDate][id]=status;
 localStorage.setItem("attendance",JSON.stringify(attendance));
 showStudents();showReport();showHistory();
}
function saveAttendance(){
 if(!students.length){alert("Add students first.");return;}
 const day=attendance[selectedDate]||{};
 const marked=students.filter(s=>day[s.id]).length;
 if(!marked){alert("Mark attendance first.");return;}
 localStorage.setItem("attendance",JSON.stringify(attendance));
 alert("✅ Attendance saved for "+formatDate(selectedDate));
 showHistory();
}
function deleteStudent(id){
 if(!confirm("Delete this student?"))return;
 students=students.filter(s=>s.id!==id);
 Object.keys(attendance).forEach(d=>{if(attendance[d])delete attendance[d][id]});
 localStorage.setItem("students",JSON.stringify(students));
 localStorage.setItem("attendance",JSON.stringify(attendance));
 showStudents();showReport();showHistory();
}
function showReport(){
 const box=document.getElementById("report");box.innerHTML="";
 if(!students.length){box.innerHTML='<div class="empty">No students yet.</div>';return;}
 students.forEach(s=>{
  let p=0,a=0;
  Object.values(attendance).forEach(day=>{
   if(day&&day[s.id]==="Present")p++;
   if(day&&day[s.id]==="Absent")a++;
  });
  const total=p+a, pct=total?((p/total)*100).toFixed(1):"0.0";
  const div=document.createElement("div");div.className="report";
  div.innerHTML=`<b>${escapeHTML(s.name)}</b><br><span class="small">${escapeHTML(s.batch)}</span><br><br>🟢 Present: ${p} &nbsp; 🔴 Absent: ${a}<br>📊 Attendance: <b>${pct}%</b>`;
  box.appendChild(div);
 });
}
function showHistory(){
 const box=document.getElementById("history");box.innerHTML="";
 const dates=Object.keys(attendance).filter(d=>attendance[d]&&Object.keys(attendance[d]).length).sort().reverse();
 if(!dates.length){box.innerHTML='<div class="empty">No saved attendance dates.</div>';return;}
 dates.forEach(d=>{
  const div=document.createElement("div");div.className="history";
  div.innerHTML=`<span>${formatDate(d)}</span><button onclick="pickDate('${d}')">Open</button>`;
  box.appendChild(div);
 });
}
function pickDate(d){
 selectedDate=d;datePicker.value=d;loadDate();
 window.scrollTo({top:0,behavior:"smooth"});
}
document.getElementById("selectedDateText").textContent="Selected: "+formatDate(selectedDate);
showStudents();showReport();showHistory();
</script>
</body>
</html>
