<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PROGRESS | Tactical Engine</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'JetBrains Mono', monospace; background-color: #050505; color: #e0e0e0; }
        .card { background: #0f0f0f; border: 1px solid #1f1f1f; border-radius: 12px; }
        .task-done { border-color: #22c55e !important; opacity: 0.5; text-decoration: line-through; }
        .btn-action { background: #1a1a1a; border: 1px solid #333; transition: all 0.2s; }
        .btn-action:active { transform: scale(0.95); background: #333; }
    </style>
</head>
<body class="max-w-md mx-auto p-4 pb-20">

    <header class="mb-6">
        <div class="flex justify-between items-end mb-2">
            <h1 class="text-xl font-bold tracking-tighter italic">PROGRESS v2.2</h1>
            <span id="rank-label" class="text-[9px] font-bold text-green-500">MANTENIMIENTO</span>
        </div>
        <div class="w-full bg-zinc-900 h-1.5 rounded-full">
            <div id="year-bar" class="bg-blue-600 h-full w-0 transition-all duration-1000"></div>
        </div>
        <p class="text-[8px] text-zinc-600 mt-1 uppercase tracking-widest text-right" id="days-left-text"></p>
    </header>

    <div class="card p-5 mb-6 flex justify-between items-center border-l-4 border-l-green-600">
        <div>
            <p class="text-[10px] text-zinc-500 uppercase">Puntos Hoy</p>
            <h2 id="main-score" class="text-5xl font-bold italic">0</h2>
        </div>
        <div class="flex gap-2">
            <button onclick="saveToJSON()" class="btn-action p-3 rounded-lg text-[10px]">💾 GUARDAR</button>
            <button onclick="resetDay()" class="btn-action p-3 rounded-lg text-[10px] text-red-400">🔄 RESET</button>
        </div>
    </div>

    <main id="tasks-root" class="space-y-6">
        </main>

    <script>
        const DATE = new Date();
        const WEEKDAY = DATE.getDay(); // 0=Dom, 1=Lun...

        // 1. DEFINICIÓN DE ACTIVIDADES (Categorías 1, 2 y 3)
        const MASTER_TASKS = {
            blocks: [ // Cat 1: Puntos Base
                { name: "5:00AM: Hidratación + Estiramiento", pts: 10, time: "AM" },
                { name: "Entrenamiento (Abdomen/Fuerza)", pts: 15, time: "AM" },
                { name: "7:30AM: Salida Z150 (Lento/Prevenido)", pts: 10, time: "AM" },
                { name: "Regreso Consciente (Seguridad)", pts: 10, time: "PM" },
                { name: "Cierre: Orden Micro-zona (10 min)", pts: 15, time: "NOCHE" },
                { name: "Desconexión Pantallas (30 min antes)", pts: 15, time: "NOCHE" }
            ],
            football: { // Cat 2: Proyecto Digital (Días clave Martes/Mier)
                1: "Scouting y Guiones Fútbol",
                2: "EDICIÓN INTENSIVA: Clip 1",
                3: "EDICIÓN INTENSIVA: Clip 2",
                4: "SEO y Portadas Shorts",
                5: "Programación Post-Producción",
                6: "Break Creativo (Enfoque Moto)",
                0: "UPLOAD: Lanzamiento Semanal"
            },
            ejes: { // Cat 3: Rumbo a los 30
                1: "Eje Físico: Técnica de Carrera",
                2: "Eje Relacional: Feedback Mariel/Serrat",
                3: "Eje Técnico: Automatización/Python",
                4: "Eje Financiero: Corte de Tarjetas/Inversión",
                5: "Eje Mental: Lectura de Largo Plazo",
                6: "Eje Mecánico: Mantto. Profundo Z150",
                0: "Eje Estratégico: Planeación Maestro"
            }
        };

        let currentData = JSON.parse(localStorage.getItem('progress_data')) || {
            score: 0,
            completed: [],
            lastUpdate: new Date().toLocaleDateString()
        };

        function init() {
            // Si es un día nuevo, resetear automáticamente
            if (currentData.lastUpdate !== new Date().toLocaleDateString()) resetDay();
            
            renderTasks();
            updateStats();
        }

        function renderTasks() {
            const root = document.getElementById('tasks-root');
            root.innerHTML = "";

            // Renderizar Bloques (Mañana, Tarde, Noche)
            const blocks = ["AM", "PM", "NOCHE"];
            blocks.forEach(b => {
                const section = document.createElement('div');
                section.innerHTML = `<h3 class="text-[10px] text-zinc-600 mb-2 uppercase tracking-widest">${b}</h3>`;
                const tasks = MASTER_TASKS.blocks.filter(t => t.time === b);
                tasks.forEach(t => section.appendChild(createTaskCard(t.name, t.pts)));
                root.appendChild(section);
            });

            // Renderizar Ejes del Día (Fútbol y 30 años)
            const specialSection = document.createElement('div');
            specialSection.innerHTML = `<h3 class="text-[10px] text-blue-500 mb-2 uppercase tracking-widest font-bold">PROYECTOS & EJES</h3>`;
            specialSection.appendChild(createTaskCard(MASTER_TASKS.football[WEEKDAY], 30));
            specialSection.appendChild(createTaskCard(MASTER_TASKS.ejes[WEEKDAY], 20));
            root.appendChild(specialSection);
        }

        function createTaskCard(name, pts) {
            const card = document.createElement('div');
            card.className = `card p-4 mb-2 flex justify-between items-center cursor-pointer ${currentData.completed.includes(name) ? 'task-done' : ''}`;
            card.innerHTML = `<span class="text-xs">${name}</span><span class="text-[9px] font-bold">+${pts}</span>`;
            card.onclick = () => toggleTask(card, name, pts);
            return card;
        }

        function toggleTask(el, name, pts) {
            if (currentData.completed.includes(name)) return;
            
            el.classList.add('task-done');
            currentData.completed.push(name);
            currentData.score += pts;
            updateStats();
            autoSave();
        }

        function updateStats() {
            document.getElementById('main-score').innerText = currentData.score;
            
            // Year Progress
            const now = new Date();
            const start = new Date(now.getFullYear(), 0, 0);
            const diff = now - start;
            const day = Math.floor(diff / (1000 * 60 * 60 * 24));
            document.getElementById('year-bar').style.width = (day/365*100) + "%";
            document.getElementById('days-left-text').innerText = `${365 - day} DÍAS PARA TUS 30`;

            const rank = document.getElementById('rank-label');
            if(currentScore >= 120) rank.innerText = "HOMBRE ÉPICO";
        }

        function autoSave() {
            localStorage.setItem('progress_data', JSON.stringify(currentData));
        }

        function saveToJSON() {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(currentData));
            const downloadAnchorNode = document.createElement('a');
            downloadAnchorNode.setAttribute("href", dataStr);
            downloadAnchorNode.setAttribute("download", "progress_backup_" + currentData.lastUpdate + ".json");
            document.body.appendChild(downloadAnchorNode);
            downloadAnchorNode.click();
            downloadAnchorNode.remove();
        }

        function resetDay() {
            currentData = { score: 0, completed: [], lastUpdate: new Date().toLocaleDateString() };
            autoSave();
            location.reload();
        }

        init();
    </script>
</body>
</html>
