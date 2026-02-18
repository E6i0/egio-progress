<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>EGIO Progress</title>

<style>
body {
  font-family: Arial, sans-serif;
  background:#0f172a;
  color:white;
  margin:0;
  padding:20px;
}

h1 {text-align:center;}

.section {
  background:#1e293b;
  padding:15px;
  margin:10px 0;
  border-radius:12px;
}

.task {
  margin:6px 0;
}

progress {
  width:100%;
  height:20px;
}

button {
  padding:8px;
  border:none;
  border-radius:8px;
  background:#38bdf8;
  cursor:pointer;
}
</style>
</head>

<body>

<h1>⚡ EGIO PROGRESS</h1>

<div class="section">
<h2>🧠 Salud + Higiene</h2>
<div id="salud"></div>
</div>

<div class="section">
<h2>🏠 Hogar del Día</h2>
<div id="hogar"></div>
</div>

<div class="section">
<h2>💻 Digital Money</h2>
<div id="digital"></div>
</div>

<div class="section">
<h2>🤝 BAMX Trabajo</h2>
<div id="bamx"></div>
</div>

<div class="section">
<h2>📊 Progreso Diario</h2>
<progress id="barra" value="0" max="100"></progress>
</div>

<script>

const tareas = {

salud: [
"Beber agua",
"Higiene personal completa",
"Movilidad ligera",
"Registrar datos smartwatch"
],

digital: [
"Revisar calendario contenido",
"Editar/publicar",
"Optimizar ideas"
],

bamx: [
"Contacto aliado",
"Seguimiento acopio",
"Planeación logística"
]

};

const hogarPorDia = {
0: ["Planificar semana", "Orden general"],
1: ["Organizar habitación"],
2: ["Limpiar cocina"],
3: ["Baño rápido"],
4: ["Área común"],
5: ["Reset general"],
6: ["Meal prep + orden"]
};

function crearChecklist(lista, contenedor, clave) {

lista.forEach((t, i) => {

let id = clave + i;

let div = document.createElement("div");
div.className = "task";

let chk = document.createElement("input");
chk.type = "checkbox";
chk.checked = localStorage.getItem(id) === "true";

chk.onchange = () => {
localStorage.setItem(id, chk.checked);
actualizarBarra();
};

div.appendChild(chk);
div.append(" " + t);

contenedor.appendChild(div);

});

}

function actualizarBarra() {

let checks = document.querySelectorAll("input[type=checkbox]");
let total = checks.length;
let activos = [...checks].filter(c => c.checked).length;

let progreso = Math.round((activos/total)*100);
document.getElementById("barra").value = progreso;

}

crearChecklist(tareas.salud, document.getElementById("salud"), "s");
crearChecklist(tareas.digital, document.getElementById("digital"), "d");
crearChecklist(tareas.bamx, document.getElementById("bamx"), "b");

let dia = new Date().getDay();
crearChecklist(hogarPorDia[dia], document.getElementById("hogar"), "h");

actualizarBarra();

</script>

</body>
</html>
