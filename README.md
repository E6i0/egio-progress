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
        .btn { background-color: var(--bg-card); border: 1px solid var(--border); transition: all 0.2s; font-size: 10px; font-weight: 700; }
        .btn-gold { border-color: var(--accent); color: var(--accent); }
        
        /* Calendario */
        .day-box { width: 10px; height: 10px; border-radius: 2px; background-color: #1a1d21; border: 1px solid #2a2d31; }
        .day-past { background-color: #2a2d31; }
        .day-active { background-color: var(--accent); box-shadow: 0 0 8px var(--accent); }

        /* Estilo para el Input Extra */
        .extra-input { border-bottom: 1px solid var(--border) !important; padding: 4px 0; font-style: italic; }
        .extra-input:focus { border-bottom-color: var(--accent) !important; }
    </style>
</head>
<body class="max-w-md mx-auto p-5 pb-24">

    <header class="mb-8 text-center">
        <h1 class="text-3xl font-extrabold tracking-tighter italic text-white">PROGRESS</h1>
        <p class="text-[9px] uppercase tracking-[0.3em] text-gray-500 mt-2">Home Order = Mental Clarity</p>
    </header>

    <section class="card p-4 mb-4">
        <div class="flex justify-between items-center mb-3">
            <p class="text-[9px] text-muted uppercase tracking-widest text-white">Rendimiento Semanal</p>
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

    <section class="card p-6 mb-6 border-l-4 border-l-[#d2a160] shadow-2xl">
        <div class="flex justify-between items-end mb-4">
            <div>
                <p class="text-[10px] text-muted uppercase tracking-widest">Score Diario</p>
                <div class="flex items-end gap-2">
                    <h2 id="main-score" class="text-6xl font-extrabold text-white">0</h2>
                    <span class="text-xs font-bold text-muted pb-1">/ 170 pts</span>
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
        <button onclick="toggleEditMode()" id="btn-edit" class="btn w-1/2 p-3 rounded-lg">✏️ EDITAR TAREAS</button>
        <button onclick="saveBackup()" class="btn w-1/2 p-3 rounded-lg">💾 RESPALDO JSON</button>
    </section>

    <main id="tasks-root" class="space-y-6"></main>

    <button onclick="addCoin()" class="fixed bottom-6 right-6 btn-gold btn w-14 h-14 rounded-full shadow-2xl flex items-center justify-center hover:scale-110 transition">
        <span class="text-2xl">🪙</span>
    </button>

    <script>
        const DATE = new Date();
        const WEEKDAY = DATE.getDay();

        let gioData = JSON.parse(localStorage.getItem('gioData_v3.2')) || {
            score: 0,
            coins: 0,
            completed: [],
            lastDate: new Date().toLocaleDateString(),
            extraTaskName: "Misión Extra del Día...", // Tarea libre
            tasks: {
                morning: [
                    { id: "m1", name: "5:00AM: Agua + Estiramiento", pts: 10 },
                    { id: "m2", name: "7:30AM: Salida Z150 (Prevenido)", pts: 10 }
                ],
                afternoon: [
                    { id: "a1", name: "BAMX / Gestión Aliados", pts: 20 },
                    { id: "a2", name: "Hidratación (1L)", pts: 10 }
                ],
                evening: [
                    { id: "e1", name: "Orden 10 min (Home Order)", pts: 15 },
                    { id: "e2", name: "Desconexión / Mental Clarity", pts: 15 }
                ]
            },
            dailyPillars: {
                2: { id: "p2", name: "EDICIÓN: Football Progress", pts: 40 }, // Martes
                6: { id: "p6", name: "Mantto. Z150 + Casa", pts: 40 },      // Sábado
                0: { id: "p0", name: "ESTRATEGIA: Plan Semanal", pts: 40 }   // Domingo
            }
        };

        let isEditMode = false;

        function init() {
            if (gioData.lastDate !== new Date().toLocaleDateString()) {
                gioData.score = 0; gioData.completed = []; gioData.lastDate = new Date().toLocaleDateString();
                gioData.extraTaskName = "Misión Extra del Día...";
            }
            renderTasks();
            updateUI();
            setupCalendar();
        }

        function setupCalendar() {
            for(let i=0; i<7; i++) {
                const dot = document.getElementById('dot-'+i);
                if (i < WEEKDAY && i !== 0) dot.classList.add('day-past');
                if (i === WEEKDAY) dot.classList.add('day-active');
            }
            const goal = new Date('2026-12-31'); 
            const diff = goal - DATE;
            document.getElementById('days-left-counter').innerText = Math.floor(diff/(1000*60*60*24)) + " DÍAS PARA TUS 30";
        }

        function renderTasks() {
            const root = document.getElementById('tasks-root');
            root.innerHTML = "";
            
            const sections = [
                { title: "🌞 Bloque Mañana", data: gioData.tasks.morning },
                { title: "🌤 Bloque Tarde", data: gioData.tasks.afternoon },
                { title: "🌙 Bloque Noche", data: gioData.tasks.evening }
            ];

            sections.forEach(s => {
                const group = document.createElement('div');
                group.innerHTML = `<h3 class="text-[9px] text-muted mb-3 uppercase tracking-widest font-bold">${s.title}</h3>`;
                s.data.forEach(t => group.appendChild(createTaskCard(t)));
                root.appendChild(group);
            });

            // SECCIÓN PILAR + EXTRA
            const pilarGroup = document.createElement('div');
            pilarGroup.className = "border-t border-zinc-800 pt-6";
            pilarGroup.innerHTML = `<h3 class="text-[9px] text-accent mb-3 uppercase tracking-widest font-bold text-white">🎯 Eje del Día & Extra</h3>`;
            
            // Pilar automático
            const pilar = gioData.dailyPillars[WEEKDAY] || { id: "p-std", name: "Foco en Metas 30s", pts: 20 };
            pilarGroup.appendChild(createTaskCard(pilar));

            // SLOT EXTRA EDITABLE
            const extraCard = document.createElement('div');
            const extraDone = gioData.completed.includes('extra-task');
            extraCard.className = `card task-item p-4 flex justify-between items-center cursor-pointer ${extraDone ? 'done' : ''}`;
            extraCard.innerHTML = `
                <div class="w-full mr-4">
                    <input type="text" value="${gioData.extraTaskName}" 
                        class="extra-input task-input text-sm italic text-accent" 
                        onchange="gioData.extraTaskName = this.value; saveData();"
                        onclick="event.stopPropagation()">
                </div>
                <span class="text-[10px] font-bold text-muted">+20</span>
            `;
            extraCard.onclick = () => {
                if(!gioData.completed.includes('extra-task')) {
                    gioData.completed.push('extra-task');
                    gioData.score += 20; updateUI(); saveData(); renderTasks();
                }
            };
            pilarGroup.appendChild(extraCard);
            
            root.appendChild(pilarGroup);
        }

        function createTaskCard(t) {
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
            return card;
        }

        function updateTaskText(id, text) {
            Object.values(gioData.tasks).forEach(cat => {
                const t = cat.find(item => item.id === id);
                if(t) t.name = text;
            });
            Object.values(gioData.dailyPillars).forEach(p => {
                if(p.id === id) p.name = text;
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
            document.getElementById('progress-bar').style.width = (gioData.score/170*100) + "%";
        }

        function addCoin() { gioData.coins++; updateUI(); saveData(); }
        function saveData() { localStorage.setItem('gioData_v3.2', JSON.stringify(gioData)); }
        function saveBackup() {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(gioData));
            const dl = document.createElement('a'); dl.setAttribute("href", dataStr);
            dl.setAttribute("download", "progress_backup.json"); dl.click();
        }

        init();
    </script>
</body>
</html>
