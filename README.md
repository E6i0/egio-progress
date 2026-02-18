<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EGIO OS | Architecture</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;600;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: #0f172a; color: #f1f5f9; }
        .glass-card { background: rgba(30, 41, 59, 0.7); backdrop-filter: blur(12px); border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 24px; }
        .current-block { border-left: 4px solid #38bdf8; background: linear-gradient(90deg, rgba(56, 189, 248, 0.1), transparent); }
        input[type="checkbox"]:checked + label { color: #64748b; text-decoration: line-through; }
    </style>
</head>
<body class="max-w-xl mx-auto p-4 pb-32">

    <header class="py-8 text-center">
        <h1 class="text-3xl font-extrabold tracking-tighter bg-gradient-to-r from-sky-400 to-indigo-400 bg-clip-text text-transparent">EGIO PROGRESS</h1>
        <p id="header-date" class="text-[10px] uppercase tracking-[0.3em] text-slate-500 mt-2 font-bold"></p>
    </header>

    <div class="glass-card p-5 mb-6 flex justify-between items-center border-b border-sky-500/20">
        <div>
            <p class="text-[10px] text-slate-400 font-bold uppercase italic">Longevidad</p>
            <p class="text-2xl font-bold text-sky-400">+<span id="longevity-score">0.0</span> <span class="text-xs font-normal text-slate-500">días</span></p>
        </div>
        <div class="text-right">
            <p class="text-[10px] text-slate-400 font-bold uppercase italic">Claridad Mental</p>
            <p id="clarity-percent" class="text-2xl font-bold text-indigo-400">0%</p>
        </div>
    </div>

    <main id="app-engine" class="space-y-4">
        </main>

    <div class="fixed bottom-6 left-1/2 -translate-x-1/2 w-[92%] max-w-lg glass-card p-4 border-t border-sky-500/30 shadow-2xl">
        <div class="flex items-center gap-3">
            <div class="flex gap-1">
                <button onclick="setMood('sad')" id="m-sad" class="text-xl grayscale hover:grayscale-0 transition-all">😢</button>
                <button onclick="setMood('neutral')" id="m-neu" class="text-xl grayscale hover:grayscale-0 transition-all">😐</button>
                <button onclick="setMood('happy')" id="m-hap" class="text-xl grayscale hover:grayscale-0 transition-all">😊</button>
            </div>
            <input type="text" id="daily-note" placeholder="Nota de hoy..." class="flex-1 bg-transparent border-b border-slate-700 text-xs py-1 outline-none focus:border-sky-500">
            <button onclick="syncToCloud()" class="bg-sky-500 text-slate-900 font-bold px-4 py-2 rounded-xl text-[10px] uppercase tracking-wider shadow-lg shadow-sky-500/20">Sync Drive</button>
        </div>
    </div>

    <script>
        const OS_PLAN = [
            { id: 'morning', time: '06:00 - 08:00', title: '🌅 Ritual Mañana', tasks: ['Tender cama', 'Lavar utensilios desayuno', 'Ventilar habitación', 'Desayuno funcional (Avena/Huevo)'] },
            { id: 'work_am', time: '08:30 - 13:00', title: '🏢 Enfoque BAMX', tasks: ['3 Tareas clave BAMX', 'Contacto aliados acopio', 'Seguimiento institucional'] },
            { id: 'work_pm', time: '14:00 - 18:00', title: '🏢 Logística & Cierre', tasks: ['Planeación eventos', 'Documentación reportes', 'Cierre escritorio físico'] },
            { id: 'egio_zone', time: '19:30 - 21:30', title: '🏠 Zona del Día', tasks: [] } // Se llena dinámicamente
        ];

        const DAILY_ZONES = {
            1: { name: "Habitación (Lunes)", tasks: ["Organizar ropa", "Limpiar superficies", "Revisar outfit semanal"] },
            2: { name: "Cocina (Martes)", tasks: ["Limpiar estufa", "Refrigerador rápido", "Ordenar despensa"] },
            3: { name: "Baño (Miércoles)", tasks: ["Lavabo y Espejo", "Sanitización rápida", "Higiene personal profunda"] },
            4: { name: "Áreas Comunes (Jueves)", tasks: ["Barrer/Trapear", "Ordenar mesa/escritorio", "Monetización/Contenido"] },
            5: { name: "Reset General (Viernes)", tasks: ["Gestionar basura", "Ropa sucia", "Evaluación Semanal EGIO"] },
            6: { name: "Producción (Sábado)", tasks: ["Batching contenido (2hrs)", "Meal Prep simple", "Compras personales"] },
            0: { name: "Plan Maestro (Domingo)", tasks: ["Revisar semana", "Preparar ropa L-V", "Planificación estratégica"] }
        };

        let mood = null;

        function initApp() {
            const now = new Date();
            const day = now.getDay();
            const hour = now.getHours();

            document.getElementById('header-date').innerText = now.toLocaleDateString('es-MX', { weekday: 'long', day: 'numeric', month: 'short' });

            // Cargar Zona del Día
            OS_PLAN[3].tasks = DAILY_ZONES[day].tasks;
            OS_PLAN[3].title = `🏠 Micro-Zona: ${DAILY_ZONES[day].name}`;

            const container = document.getElementById('app-engine');
            container.innerHTML = OS_PLAN.map(block => {
                const startHour = parseInt(block.time.split(':')[0]);
                const isCurrent = hour >= startHour && hour < (startHour + 4);

                return `
                    <div class="glass-card p-5 ${isCurrent ? 'current-block ring-1 ring-sky-500/30' : 'opacity-50'}">
                        <div class="flex justify-between items-start mb-3">
                            <div>
                                <h3 class="text-[10px] font-bold text-sky-400 uppercase tracking-tighter">${block.time}</h3>
                                <p class="font-bold text-md">${block.title}</p>
                            </div>
                        </div>
                        <div class="space-y-2">
                            ${block.tasks.map(task => {
                                const taskId = btoa(task).substring(0,10); // ID único basado en texto
                                return `
                                <div class="flex items-center">
                                    <input type="checkbox" id="${taskId}" onchange="saveCheck('${taskId}')" ${getCheck(taskId)} class="w-4 h-4 rounded bg-slate-800 border-slate-700 text-sky-500 focus:ring-0">
                                    <label for="${taskId}" class="ml-3 text-xs font-medium text-slate-300 transition-all">${task}</label>
                                </div>
                            `}).join('')}
                        </div>
                    </div>
                `;
            }).join('');
            updateStats();
        }

        function saveCheck(id) {
            localStorage.setItem('egio_t_'+id, document.getElementById(id).checked);
            updateStats();
        }

        function getCheck(id) {
            return localStorage.getItem('egio_t_'+id) === 'true' ? 'checked' : '';
        }

        function updateStats() {
            const checks = document.querySelectorAll('input[type="checkbox"]');
            const done = Array.from(checks).filter(c => c.checked).length;
            const percent = checks.length > 0 ? Math.round((done / checks.length) * 100) : 0;
            document.getElementById('clarity-percent').innerText = percent + '%';
            document.getElementById('longevity-score').innerText = (done * 0.05).toFixed(1);
        }

        function setMood(m) {
            mood = m;
            document.querySelectorAll('button[id^="m-"]').forEach(b => b.classList.add('grayscale'));
            document.getElementById('m-' + (m === 'happy' ? 'hap' : m === 'sad' ? 'sad' : 'neu')).classList.remove('grayscale');
        }

        async function syncToCloud() {
            if(!mood) return alert("Selecciona un estado emocional");
            const data = { date: new Date().toLocaleString(), mood: mood, note: document.getElementById('daily-note').value, clarity: document.getElementById('clarity-percent').innerText };
            
            const DRIVE_URL = 'TU_URL_DE_APPS_SCRIPT'; // <--- Pon tu URL aquí

            try {
                await fetch(DRIVE_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(data)});
                alert("Sincronización Exitosa. Checks reseteados para mañana.");
                localStorage.clear();
                location.reload();
            } catch (e) { alert("Guardado local solamente."); }
        }

        initApp();
    </script>
</body>
</html>
