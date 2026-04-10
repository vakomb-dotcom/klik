<!DOCTYPE html>
<html lang="lt">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>10K MISIJA</title>

<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>

body{
margin:0;
font-family:'Inter',sans-serif;
background:linear-gradient(135deg,#eef2ff,#f8fafc);
color:#111;
}

.container{
max-width:700px;
margin:auto;
padding:20px;
}

.topbar{
display:flex;
justify-content:space-between;
align-items:center;
margin-bottom:10px;
}

.menuBtn{
background:#111;
color:white;
padding:8px 14px;
border-radius:10px;
cursor:pointer;
}

.menu{
display:none;
background:white;
padding:15px;
border-radius:16px;
margin-bottom:15px;
box-shadow:0 10px 25px rgba(0,0,0,0.1);
}

h1{
text-align:center;
color:#16a34a;
font-size:34px;
font-weight:800;
margin-bottom:10px;
}

.card{
background:rgba(255,255,255,0.9);
padding:20px;
border-radius:20px;
margin-top:20px;
box-shadow:0 15px 35px rgba(0,0,0,0.08);
}

.card-header{
display:flex;
justify-content:space-between;
align-items:center;
margin-bottom:10px;
}

.progress{
height:14px;
background:#e5e7eb;
border-radius:20px;
overflow:hidden;
margin-top:10px;
}

.fill{
height:100%;
background:linear-gradient(90deg,#22c55e,#4ade80);
}

.big{
font-size:26px;
font-weight:800;
margin-top:10px;
}

.week{
color:#2563eb;
font-size:15px;
}

input, textarea{
width:100%;
padding:12px;
margin-top:10px;
border-radius:12px;
border:1px solid #e5e7eb;
font-size:14px;
background:#f9fafb;
}

button{
width:100%;
padding:12px;
margin-top:10px;
border:none;
border-radius:12px;
background:linear-gradient(90deg,#22c55e,#16a34a);
color:white;
font-weight:600;
cursor:pointer;
}

.toggleBtn{
background:#111;
font-size:12px;
padding:6px 10px;
border-radius:10px;
width:auto;
color:white;
}

.log{
margin-top:12px;
background:#f1f5f9;
padding:12px;
border-radius:12px;
max-height:200px;
overflow:auto;
font-size:13px;
display:none;
}

/* 🔥 FIXED TASK BLOCK */
.task{
display:flex;
align-items:center;
justify-content:space-between;
gap:10px;
background:#f8fafc;
padding:12px;
border-radius:12px;
margin-top:10px;
border:1px solid #e5e7eb;
}

.task-left{
display:flex;
align-items:center;
gap:10px;
flex:1;
min-width:0;
}

.task-text{
flex:1;
min-width:0;
display:block;
white-space:normal;
word-break:break-word;
font-size:15px;
}

/* DONE */
.task.done{
background:#dcfce7;
text-decoration:line-through;
}

.delete{
background:#ef4444;
color:white;
border:none;
padding:6px 10px;
border-radius:8px;
cursor:pointer;
width:auto;
flex-shrink:0;
}

.reset{
background:#ef4444;
}

.stat{
margin-top:10px;
display:flex;
justify-content:space-between;
align-items:center;
}

.stat button{
width:auto;
padding:6px 10px;
margin-left:10px;
}

</style>
</head>

<body>

<div class="container">

<div class="topbar">
<h1>🎯 10K MISIJA</h1>
<div class="menuBtn" onclick="toggle('menu')">☰ MENU</div>
</div>

<div id="menu" class="menu">

<h3>🚗 STATISTIKA</h3>

<div class="stat">
<span>Nupirkta: <b id="boughtText">0</b></span>
<button onclick="bought++ ; save(); render()">+</button>
</div>

<div class="stat">
<span>Parduota: <b id="soldText">0</b></span>
<button onclick="sold++ ; save(); render()">+</button>
</div>

<h3>📊 PELNAS PER DIENAS</h3>
<canvas id="chart"></canvas>

</div>

<div class="card">
<div class="card-header">
<h3>💰 PELNAS</h3>
<button class="toggleBtn" onclick="toggle('profitLog')">📜 Log</button>
</div>

<input id="profitAmount" type="number">
<textarea id="profitDesc"></textarea>

<button onclick="addProfit()">➕ Pridėti pelną</button>

<div class="progress"><div id="profitBar" class="fill"></div></div>

<p class="big" id="profitText"></p>
<p class="week" id="weekProfit"></p>

<div id="profitLog" class="log"></div>
</div>

<div class="card">
<div class="card-header">
<h3>📋 VEIKSMAI</h3>
<button class="toggleBtn" onclick="toggle('taskLog')">📜 Log</button>
</div>

<input id="taskInput">
<button onclick="addTask()">➕ Pridėti veiksmą</button>

<div id="taskList"></div>
<div id="taskLog" class="log"></div>
</div>

<button class="reset" onclick="resetAll()">❌ RESET</button>

</div>

<script>

let profit=0, profitLogs=[], tasks=[], taskLogs=[];
let lastDate="", bought=0, sold=0;
let chart;

function today(){return new Date().toDateString();}
function now(){return new Date().toLocaleString();}

function checkDay(){
if(lastDate!==today()){
tasks = tasks.filter(t=>!t.done);
lastDate = today();
save();
}
}

function addProfit(){
let val=parseInt(profitAmount.value);
if(!val) return;

profit+=val;
profitLogs.unshift(`${now()} — ${profitDesc.value} +${val}€`);

profitAmount.value="";
profitDesc.value="";

save(); render(); updateChart();
}

function addTask(){
let text=taskInput.value.trim();
if(!text) return;

tasks.push({text,done:false});
taskInput.value="";

save(); render();
}

function toggleTask(i){
tasks[i].done=!tasks[i].done;

if(tasks[i].done){
taskLogs.unshift(`${now()} — ✔️ ${tasks[i].text}`);
}

save(); render();
}

function deleteTask(i){
tasks.splice(i,1);
save(); render();
}

function calcWeek(){
let nowDate=new Date();
let weekAgo=new Date();
weekAgo.setDate(nowDate.getDate()-7);

let sum=0;
profitLogs.forEach(l=>{
let d=new Date(l.split(" — ")[0]);
if(d>=weekAgo){
let val=parseInt(l.split("+")[1]);
if(!isNaN(val)) sum+=val;
}
});
return sum;
}

function updateChart(){
let map={};
profitLogs.forEach(l=>{
let date=l.split(",")[0];
let val=parseInt(l.split("+")[1]);
map[date]=(map[date]||0)+val;
});

let labels=Object.keys(map);
let data=Object.values(map);

if(chart) chart.destroy();

chart=new Chart(document.getElementById("chart"),{
type:'line',
data:{labels,datasets:[{data,borderColor:'#22c55e',fill:false}]}
});
}

function toggle(id){
let el=document.getElementById(id);
el.style.display=el.style.display==="block"?"none":"block";
}

function render(){

checkDay();

profitBar.style.width=Math.min(profit/10000*100,100)+"%";
profitText.innerText=`${profit}€ / 10 000€`;
weekProfit.innerText=`📅 7 dienos: ${calcWeek()}€`;

profitLog.innerHTML=profitLogs.join("<br>");
taskLog.innerHTML=taskLogs.join("<br>");

boughtText.innerText=bought;
soldText.innerText=sold;

taskList.innerHTML="";

tasks.forEach((t,i)=>{
let div=document.createElement("div");
div.className="task"+(t.done?" done":"");

let left=document.createElement("div");
left.className="task-left";

let checkbox=document.createElement("input");
checkbox.type="checkbox";
checkbox.checked=t.done;
checkbox.onclick=()=>toggleTask(i);

let text=document.createElement("div");
text.className="task-text";
text.textContent=t.text;

left.appendChild(checkbox);
left.appendChild(text);

let del=document.createElement("button");
del.className="delete";
del.textContent="X";
del.onclick=()=>deleteTask(i);

div.appendChild(left);
div.appendChild(del);

taskList.appendChild(div);
});

}

function save(){
localStorage.setItem("profit",profit);
localStorage.setItem("profitLogs",JSON.stringify(profitLogs));
localStorage.setItem("tasks",JSON.stringify(tasks));
localStorage.setItem("taskLogs",JSON.stringify(taskLogs));
localStorage.setItem("bought",bought);
localStorage.setItem("sold",sold);
localStorage.setItem("lastDate",lastDate);
}

function load(){
profit=parseInt(localStorage.getItem("profit"))||0;
profitLogs=JSON.parse(localStorage.getItem("profitLogs"))||[];
tasks=JSON.parse(localStorage.getItem("tasks"))||[];
taskLogs=JSON.parse(localStorage.getItem("taskLogs"))||[];
bought=parseInt(localStorage.getItem("bought"))||0;
sold=parseInt(localStorage.getItem("sold"))||0;
lastDate=localStorage.getItem("lastDate")||today();
}

function resetAll(){
if(!confirm("Tikrai ištrinti viską?")) return;
localStorage.clear();
location.reload();
}

load();
render();
updateChart();

</script>

</body>
</html>
