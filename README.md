<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>EGIO Progress</title>

<script src="https://cdn.tailwindcss.com"></script>

<style>
body {
    font-family: system-ui;
    background:#0f172a;
    color:#f1f5f9;
}

.glass-card {
    background: rgba(30,41,59,0.7);
    border-radius:16px;
    border:1px solid rgba(255,255,255,0.1);
}

input[type="checkbox"]:checked + label {
    text-decoration: line-through;
    color:#64748b;
}
</style>
</head>

<body class="max-w-xl mx-auto p-4 pb-32">

<header class="py-6 text-center">
<h1 class="text-3xl font-bold text-sky-400">EGIO PROGRESS</h1>
<p id="header-date" class="text-xs text-slate-400 mt-1"></p>
</header>

<div class="glass-card p-3 mb-4 flex justify-between">
<span class="text-xs">Fecha:</span>
<input type="date" id="date-picker"
class="bg-slate-800 text-xs p-1 rounded">
</div>

<div class="glass-card p-4 mb-4 flex justify-between">
<div>
<p class="text-xs text-slate-400">Longevidad</p>
<p class="text-xl text-sky-400">
+<span id="longevity-score">0</span> días
</p>
</div>

<div>
<p class="text-xs text-slate-400">Claridad</p>
<p id="clarity-percent" class="text-xl text-indigo-400">0%</p>
</div>
</div>

<main id="app-engine" class="space-y-4"></main>

<div class="fixed bottom-4 left-1/2 -translate-x-1/2 w-[95%] max-w-lg glass-card p-4">

<div class="flex gap-2 mb-2">
<button onclick="setMood('sad')">😢</button>
<button onclick="setMood('neutral')">😐</button>
<button onclick="setMood('happy')">😊</button>
</div>

<input id="daily-note"
placeholder="Nota del día..."
class="w-full bg-slate-800 p-2 text-xs rounded">

</div>

<script>

const OS_PLAN = [
{
title:"🌅 Ritual Mañana",
tasks:[
"Tender cama",
"Lavar utensilios",
"Ventilar habitación",
"Desayuno funcional"
]
},
{
title:"🏢 Trabajo BAMX",
tasks:[
"Tarea clave 1",
"Tarea clave 2",
"Seguimiento aliado"
]
},
{
title:"🏠 Zona del día",
tasks:[
"Organizar espacio",
"Limpieza rápida",
"Preparación personal"
]
},
{
title:"💻 Digital",
tasks:[
"Revisar contenido",
"Optimizar ideas",
"Programar publicaciones"
]
}
];

let mood = null;
let currentKey = "";

function getDateKey(date){
return "egio_" + date.toISOString().split("T")[0];
}

function buildUI(){

const container = document.getElementById("app-engine");

container.innerHTML = OS_PLAN.map(section => `

<div class="glass-card p-4">

<h3 class="font-bold mb-2">${section.title}</h3>

${section.tasks.map(task => {

const id = btoa(section.title+task).slice(0,12);

return `
<div class="flex items-center mb-1">
<input type="checkbox"
id="${id}"
onchange="saveDay()"
class="mr-2">
<label for="${id}" class="text-xs">${task}</label>
</div>
`;

}).join("")}

</div>

`).join("");

}

function loadDay(){

const picker = document.getElementById("date-picker");
const date = new Date(picker.value);

currentKey = getDateKey(date);

const data = JSON.parse(localStorage.getItem(currentKey)) || {
checks:{},
note:"",
mood:null
};

mood = data.mood;

document.querySelectorAll("input[type=checkbox]")
.forEach(c => c.checked = data.checks[c.id] || false);

document.getElementById("daily-note").value = data.note;

updateStats();

}

function saveDay(){

const checks = {};

document.querySelectorAll("input[type=checkbox]")
.forEach(c => checks[c.id] = c.checked);

const data = {
checks,
note: document.getElementById("daily-note").value,
mood
};

localStorage.setItem(currentKey, JSON.stringify(data));

updateStats();

}

function updateStats(){

const checks = document.querySelectorAll("input[type=checkbox]");
const done = [...checks].filter(c=>c.checked).length;

const percent = checks.length
? Math.round(done/checks.length*100)
:0;

document.getElementById("clarity-percent").innerText = percent+"%";
document.getElementById("longevity-score").innerText =
(done*0.05).toFixed(1);

}

function setMood(m){

mood = m;
saveDay();

}

function init(){

buildUI();

const today = new Date();
document.getElementById("header-date")
.innerText = today.toLocaleDateString();

const picker = document.getElementById("date-picker");
picker.value = today.toISOString().split("T")[0];

picker.addEventListener("change", loadDay);

document.getElementById("daily-note")
.addEventListener("input", saveDay);

loadDay();

}

init();

</script>

</body>
</html>
