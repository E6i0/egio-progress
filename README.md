<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EGIO PROGRESS | Sistema Operativo Personal</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f3f4f6; color: #1f2937; }
        .card { background: white; border-radius: 16px; padding: 1.25rem; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1); }
        .task-done { text-decoration: line-through; opacity: 0.5; }
        .btn-status { transition: all 0.2s; filter: grayscale(1); font-size: 1.5rem; }
        .btn-status.active { filter: grayscale(0); transform: scale(1.2); }
    </style>
</head>
<body class="max-w-2xl mx-auto p-4 pb-24">

    <header class="flex justify-between items-end my-6">
        <div>
            <h1 class="text-2xl font-extrabold text-slate-900 tracking-tight">EGIO <span class="text-blue-600">OS</span></h1>
            <p id="current-date" class="text-xs text-slate-500 font-bold uppercase tracking-widest"></p>
        </div>
        <div class="text-right">
            <div class="text-[10px] font-bold text-slate-400 uppercase">Progreso del Día</div>
            <div class="w-24 bg-gray-200 h-2 rounded-full mt-1">
                <div id="day-progress-bar" class="bg-emerald-500 h-2 rounded-full transition-all" style="width: 0%"></div>
            </div>
        </div>
    </header>

    <section class="mb-6">
        <h2 class="text-sm font-bold text-blue-600 uppercase mb-3 tracking-tighter">⚡ Foco Actual</h2>
        <div id="current-focus-card" class="card border-l-4 border-blue-500">
            </div>
    </section>

    <section class="mb-6">
        <h2 class="text-sm font-bold text-slate-500 uppercase mb-3 tracking-tighter italic">📝 Tareas Clave</h2>
        <div id="tasks-list" class="space-y-2">
            </div>
    </section>

    <section class="card border-t-4 border-slate-900">
        <h3 class="text-sm font-bold mb-4 uppercase">Cierre de Jornada</h3>
        <div class="flex justify-between items-center mb-4">
            <div class="flex gap-4">
                <button onclick="setStatus('sad')" id="btn-sad" class="btn-status">😢</button>
                <button onclick="setStatus('neutral')" id="btn-neutral" class="btn-status">😐</button>
                <button onclick="setStatus('happy')" id="btn-happy" class="btn-status">😊</button>
            </div>
            <input type="text" id="day-note" placeholder="Reseña breve..." class="flex-1 ml-4 border-b border-slate-300 outline-none text-sm">
        </div>
        <button onclick="saveDay()" class="w-full bg-slate-900 text-white py-3 rounded-xl font-bold text-sm">GUARDAR EN LOGS</button>
    </section>

    <section id="history" class="mt-8 space-y-2">
        <h3 class="text-[10px] font-bold text-slate-400 uppercase tracking-widest">Historial de Cumplimiento</h3>
        </section>

    <script>
        const schedule = [
            { start: 6, end: 8, title: "🌅 Mañana: Higiene & Traslado", tasks: ["Tender cama", "Outfit listo", "Audio educativo/Reflexión"] },
            { start: 8, end: 13, title: "🏢 Trabajo: Bloque BAMX", tasks: ["3 Tareas clave BAMX", "Contacto aliados", "Seguimiento acopio"] },
            { start: 13, end: 18, title: "🏢 Trabajo: Logística", tasks: ["Planeación evento", "Documentación", "Cierre jornada"] },
            { start: 18, end: 20, title: "🚗 Regreso & EGIO Bloque", tasks: ["Desconexión mental", "Micro-zona del día"] },
            { start: 20, end: 24, title: "🌙 Cierre & Descanso", tasks: ["Higiene personal", "Revisión biométrica", "Plan día siguiente"] }
        ];

        const daySpecifics = {
            1: "Lunes: Salud + Revisión Biométrica",
            2: "Martes: Producción Contenido Digital",
            3: "Miércoles: Higiene Profunda Personal",
            4: "Jueves: Monetización / Programación",
            5: "Viernes: Evaluación Semanal EGIO",
            6: "Sábado: Batching Contenido (2hrs)",
            0: "Domingo: Planificación Semanal"
        };

        let currentStatus = null;
        let logs = JSON.parse(localStorage.getItem('egio_logs')) || [];

        function init() {
            const now = new Date();
            const hour = now.getHours();
            const day = now.getDay();
            
            document.getElementById('current-date').innerText = now.toLocaleDateString('es-MX', { weekday: 'short', day: 'numeric', month: 'short' });

            // Cargar Foco Actual
            const currentSlot = schedule.find(s => hour >= s.start && hour < s.end) || schedule[0];
            const dayTheme = daySpecifics[day];
            
            document.getElementById('current-focus-card').innerHTML = `
                <p class="text-xs font-bold text-blue-400 uppercase tracking-wider">${currentSlot.title}</p>
                <p class="font-bold text-lg">${dayTheme}</p>
            `;

            // Cargar Checkbox con las tareas del slot actual
            const list = document.getElementById('tasks-list');
            list.innerHTML = currentSlot.tasks.map((t, i) => `
                <div class="flex items-center bg-white p-3 rounded-xl shadow-sm border border-slate-100">
                    <input type="checkbox" onchange="updateProgress()" class="w-5 h-5 rounded-full border-2 border-blue-500 accent-blue-500">
                    <span class="ml-3 text-sm font-medium text-slate-700">${t}</span>
                </div>
            `).join('');

            renderLogs();
        }

        function updateProgress() {
            const checks = document.querySelectorAll('input[type="checkbox"]');
            const done = Array.from(checks).filter(c => c.checked).length;
            const percent = (done / checks.length) * 100;
            document.getElementById('day-progress-bar').style.width = percent + '%';
        }

        function setStatus(s) {
            currentStatus = s;
            document.querySelectorAll('.btn-status').forEach(b => b.classList.remove('active'));
            document.getElementById('btn-' + s).classList.add('active');
        }

        function saveDay() {
            if(!currentStatus) return alert("Selecciona una carita");
            const entry = {
                id: Date.now(),
                date: new Date().toLocaleString('es-MX', {hour:'2-digit', minute:'2-digit', day:'2-digit', month:'2-digit'}),
                status: currentStatus,
                note: document.getElementById('day-note').value,
                longevity: (Math.random() * (0.8 - 0.1) + 0.1).toFixed(2) // Simulación de ganancia
            };
            logs.unshift(entry);
            localStorage.setItem('egio_logs', JSON.stringify(logs));
            renderLogs();
            // Reset UI
            document.getElementById('day-note').value = "";
            setStatus(null);
        }

        function renderLogs() {
            document.getElementById('history').innerHTML = '<h3 class="text-[10px] font-bold text-slate-400 uppercase tracking-widest">Historial</h3>' + 
            logs.map(l => `
                <div class="flex justify-between items-center bg-white p-3 rounded-lg text-xs shadow-sm">
                    <span>${l.status === 'happy' ? '😊' : l.status === 'neutral' ? '😐' : '😢'} <b>${l.date}</b> - ${l.note}</span>
                    <span class="text-emerald-600 font-bold">+${l.longevity}d</span>
                </div>
            `).join('');
        }

        init();
    </script>
</body>
</html>
