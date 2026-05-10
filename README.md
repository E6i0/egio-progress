<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PROGRESS | Executive Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;500;700;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-main: #0c0d0e;
            --bg-card: #151719;
            --border: #2a2d31;
            --text-main: #e4e6eb;
            --text-muted: #8b949e;
            --accent: #d2a160;
            --success: #2ea043;
        }

        body { font-family: 'Inter', sans-serif; background-color: var(--bg-main); color: var(--text-main); }
        .card { background-color: var(--bg-card); border: 1px solid var(--border); border-radius: 12px; }
        .task-item { transition: all 0.2s ease; border-left: 2px solid transparent; }
        .task-item.done { border-left-color: var(--success); opacity: 0.5; text-decoration: line-through; }
        .task-input { background: transparent; border: none; color: inherit; width: 100%; outline: none; }
        .progress-fill { transition: width 0.8s cubic-bezier(0.4, 0, 0.2, 1); background-color: var(--accent); }
        .btn { background-color: var(--bg-card); border: 1px solid var(--border); transition: all 0.2s; }
        .btn-gold { border-color: var(--accent); color: var(--accent); }
        
        /* Estilo Calendario Semanal */
        .day-box { width: 10px; height: 10px; border-radius: 2px; background-color: #1a1d21; border: 1px solid #2a2d31; }
        .day-past { background-color: #2a2d31; }
        .day-active { background-color: var(--accent); box-shadow: 0 0 8px var(--accent); }
    </style>
</head>
<body class="max-w-md mx-auto p-5 pb-24">

    <header class="mb-8 text-center">
        <h1 class="text-3xl font-extrabold tracking-tighter italic text-white italic">PROGRESS</h1>
        <p class="text-[9px] uppercase tracking-[0.3em] text-gray-500 mt-2">Home Order = Mental Clarity</p>
    </header>

    <section class="card p-4 mb-4">
        <div class="flex justify-between items-center mb-3">
            <p class="text-[9px] text-muted uppercase tracking-widest">Rendimiento Semanal</p>
            <p id="days-left-counter" class="text-[9px] text-accent font-bold uppercase"></p>
        </div>
        <div class="flex justify-between px-2" id="week-dots">
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="dot-1"></div><span class="text-[7px]">L</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="dot-2"></div><span class="text-[7px]">M</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="dot-3"></div><span class="text-[7px]">M</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="dot-4"></div><span class="text-[7px]">J</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="dot-5"></div><span class="text-[7px]">V</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="dot-6"></div><span class="text-[7px]">S</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="dot-0"></div><span class="text-[7px]">D</span></div>
        </div>
    </section>

    <section class="card p-6 mb-6 border-l-4 border-l-[#d2a160]">
        <div class="flex justify-between items-end mb-4">
            <div>
                <p class="text-[10px] text-muted uppercase tracking-widest">Score Diario</p>
                <div class="flex items-end gap-2">
                    <h2 id="main-score" class="text-6xl font-extrabold text-white">0</h2>
                    <span class="text-xs font-bold text-muted pb-1">/ 150</span>
                </div>
            </div>
            <div class="text-right">
                <p class="text-[10px] text-muted uppercase tracking-widest">Sobriedad</p>
                <div class="flex items-center gap-1 text-accent justify-end">
                    <span id="coin-count" class="text-xl font-bold">0</span><span>🪙</span>
                </div>
            </div>
        </div>
        <div class="w-full bg-zinc-800 h-1.5 rounded-full overflow-hidden">
            <div id="progress-bar" class="progress-fill h-full w-0"></div>
        </div>
    </section>

    <section class="flex gap-2 mb-8">
        <button onclick="toggleEditMode()" id="btn-edit" class="btn w-1/2 p-3 rounded-lg text-[10px] font-bold">✏️ EDITAR TAREAS</button>
        <button onclick="saveBackup()" class="btn w-1/2 p-3 rounded-lg text-[10px] font-bold">💾 RESPALDO JSON</button>
    </section>

    <main id="tasks-root" class="space-y-6"></main>

    <button onclick="addCoin()" class="fixed bottom-6 right-6 btn-gold btn w-14 h-14 rounded-full shadow-2xl flex items-center justify-center hover:scale-110 transition">
        <span class="text-2xl">🪙</span>
    </button>

    <script>
        const DATE = new Date();
        const WEEKDAY = DATE.getDay();

        let gioData = JSON.parse(localStorage.getItem('gioData_v3.1')) || {
            score: 0,
            coins: 0,
            completed: [],
            lastDate: new Date().toLocaleDateString(),
            tasks: {
                morning: [
                    { id: "m1", name: "5:00AM: Agua + Estiramiento", pts: 10 },
                    { id: "m2", name: "Entrenamiento Fuerza/Abs", pts: 15 },
                    { id: "m3", name: "7:30AM: Salida Z150 (Prevenido)", pts: 10 }
                ],
                afternoon: [
                    { id: "a1", name: "Aliados BAMX / Trabajo", pts: 20 },
                    { id: "a2", name: "Hidratación (1L)", pts: 10 }
                ],
                evening: [
                    { id: "e1", name: "Orden Espacio (10 min)", pts: 10 },
                    { id: "e2", name: "Edición / Football Progress", pts: 30 },
                    { id: "e3", name: "Desconexión / Sueño", pts: 15 }
                ]
            }
        };

        let isEditMode = false;

        function init() {
            if (gioData.lastDate !== new Date().toLocaleDateString()) {
                gioData.score = 0; gioData.completed = []; gioData.lastDate = new Date().toLocaleDateString();
            }
            renderTasks();
            updateUI();
            setupCalendar();
        }

        function setupCalendar() {
            for(let i=0; i<7; i++) {
                const dot = document.getElementById('dot-'+i);
                if (i < WEEKDAY && i !== 0) dot.classList.add('day-past'); // Lunes a Viernes pasados
                if (i === WEEKDAY) dot.classList.add('day-active');
            }
            // Días para los 30 (Asumiendo 2026 como año clave)
            const goal = new Date('2026-12-31'); 
            const diff = goal - DATE;
            document.getElementById('days-left-counter').innerText = Math.floor(diff/(1000*60*60*24)) + " DÍAS RESTANTES";
        }

        function renderTasks() {
            const root = document.getElementById('tasks-root');
            root.innerHTML = "";
            
            const groups = [
                { title: "🌞 Mañana", data: gioData.tasks.morning },
                { title: "🌤 Tarde", data: gioData.tasks.afternoon },
                { title: "🌙 Noche", data: gioData.tasks.evening }
            ];

            groups.forEach(g => {
                const section = document.createElement('div');
                section.innerHTML = `<h3 class="text-[9px] text-muted mb-3 uppercase tracking-widest font-bold">${g.title}</h3>`;
                g.data.forEach(t => {
                    const card = document.createElement('div');
                    const done = gioData.completed.includes(t.id);
                    card.className = `card task-item p-4 mb-2 flex justify-between items-center cursor-pointer ${done ? 'done' : ''}`;
                    
                    if (isEditMode) {
                        card.innerHTML = `<input type="text" value="${t.name}" onblur="updateTaskText('${t.id}', this.value)" class="task-input text-sm">`;
                    } else {
                        card.innerHTML = `<span class="text-sm font-medium">${t.name}</span><span class="text-[10px] font-bold text-muted">+${t.pts}</span>`;
                        card.onclick = () => {
                            if(!gioData.completed.includes(t.id)) {
                                gioData.completed.push(t.id);
                                gioData.score += t.pts;
                                updateUI(); saveData(); renderTasks();
                            }
                        };
                    }
                    section.appendChild(card);
                });
                root.appendChild(section);
            });
        }

        function updateTaskText(id, text) {
            Object.values(gioData.tasks).forEach(cat => {
                const t = cat.find(item => item.id === id);
                if(t) t.name = text;
            });
        }

        function toggleEditMode() {
            isEditMode = !isEditMode;
            document.getElementById('btn-edit').innerText = isEditMode ? "💾 GUARDAR" : "✏️ EDITAR";
            renderTasks();
            if(!isEditMode) saveData();
        }

        function updateUI() {
            document.getElementById('main-score').innerText = gioData.score;
            document.getElementById('coin-count').innerText = gioData.coins;
            document.getElementById('progress-bar').style.width = (gioData.score/150*100) + "%";
        }

        function addCoin() { gioData.coins++; updateUI(); saveData(); }
        function saveData() { localStorage.setItem('gioData_v3.1', JSON.stringify(gioData)); }
        function saveBackup() {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(gioData));
            const dl = document.createElement('a');
            dl.setAttribute("href", dataStr);
            dl.setAttribute("download", "progress_backup.json");
            dl.click();
        }

        init();
    </script>
</body>
</html>
