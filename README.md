<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PROGRESS | Executive Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;500;700;800&display=swap" rel="stylesheet">
    <style>
        /* PALETA DE COLORES ELEGANTE */
        :root {
            --bg-main: #0c0d0e;     /* Negro Ejecutivo */
            --bg-card: #151719;     /* Zinc Oscuro */
            --border: #2a2d31;      /* Gris Borde */
            --text-main: #e4e6eb;   /* Platino */
            --text-muted: #8b949e;  /* Gris Tenue */
            --accent: #d2a160;      /* Oro Viejo */
            --success: #2ea043;     /* Verde Esmeralda */
        }

        body { font-family: 'Inter', sans-serif; background-color: var(--bg-main); color: var(--text-main); }
        .card { background-color: var(--bg-card); border: 1px solid var(--border); border-radius: 12px; }
        .border-accent { border-color: var(--accent); }
        
        /* Estilo Tarea Completada */
        .task-item { transition: all 0.2s ease; border-left: 2px solid transparent; }
        .task-item:hover { background-color: #1a1d21; }
        .task-item.done { border-left-color: var(--success); opacity: 0.5; text-decoration: line-through; }

        /* Input de Edición invisible hasta el hover */
        .task-input { background: transparent; border: none; color: inherit; width: 100%; outline: none; }
        .task-input:focus { color: var(--accent); }

        /* Barra de Progreso Oro Vieja */
        .progress-fill { transition: width 0.8s cubic-bezier(0.4, 0, 0.2, 1); background-color: var(--accent); }
        
        /* Botones Elegantes */
        .btn { background-color: var(--bg-card); border: 1px solid var(--border); transition: all 0.2s; }
        .btn:active { transform: scale(0.98); background-color: var(--border); }
        .btn-gold { border-color: var(--accent); color: var(--accent); }
    </style>
</head>
<body class="max-w-md mx-auto p-5 pb-24">

    <header class="mb-10 text-center">
        <h1 class="text-3xl font-extrabold tracking-tighter italic text-white" style="font-style: italic;">PROGRESS</h1>
        <p id="days-left" class="text-[9px] uppercase tracking-[0.3em] text-gray-500 mt-2">Home Order = Mental Clarity</p>
    </header>

    <section class="card p-6 mb-8 border-l-4 border-accent shadow-xl">
        <div class="flex justify-between items-end mb-5">
            <div>
                <p class="text-[10px] text-muted uppercase font-medium tracking-widest">Puntos Acumulados</p>
                <div class="flex items-end gap-2">
                    <h2 id="main-score" class="text-7xl font-extrabold text-white">0</h2>
                    <span class="text-sm font-bold text-muted pb-1">/ 150 pts</span>
                </div>
            </div>
            <div class="text-right">
                <p class="text-[10px] text-muted uppercase font-medium tracking-widest">Sobriedad</p>
                <div class="flex items-center gap-1.5 text-accent justify-end">
                    <span id="coin-count" class="text-xl font-bold">0</span>
                    <span class="text-xl">🪙</span>
                </div>
            </div>
        </div>
        
        <div class="w-full bg-zinc-800 h-2 rounded-full overflow-hidden">
            <div id="progress-bar" class="progress-fill h-full w-0"></div>
        </div>
    </section>

    <section class="flex gap-2 mb-8">
        <button onclick="toggleEditMode()" id="btn-edit" class="btn w-1/2 p-3 rounded-lg text-xs font-medium">✏️ EDITAR TAREAS</button>
        <button onclick="saveBackup()" class="btn w-1/2 p-3 rounded-lg text-xs font-medium">💾 GUARDAR RESPALDO</button>
    </section>

    <main id="tasks-root" class="space-y-6">
        </main>

    <button onclick="addSobrietyCoin()" class="fixed bottom-6 right-6 btn-gold btn w-16 h-16 rounded-full shadow-2xl flex items-center justify-center hover:scale-110 transition">
        <span class="text-3xl">🪙</span>
    </button>

    <script>
        const DATE = new Date();
        const WEEKDAY = DATE.getDay(); // 0=Dom, 1=Lun...

        // 1. TAREAS MAESTRAS (Editable por el usuario)
        let DEFAULT_TASKS = {
            morning: [
                { id: "m1", name: "Hacer la cama", pts: 10 },
                { id: "m2", name: "Higiene Care", pts: 10 },
                { id: "m3", name: "Deep Work (10 min)", pts: 15 },
                { id: "m4", name: "Intención Clara (MIT)", pts: 15 }
            ],
            afternoon: [
                { id: "a1", name: "Contactar 3 Aliados BAMX", pts: 20 },
                { id: "a2", name: "Hidratación (1L consciente)", pts: 10 },
                { id: "a3", name: "Check-in Emocional", pts: 10 }
            ],
            evening: [
                { id: "e1", name: "Orden Espacio 10 min", pts: 10 },
                { id: "e2", name: "Ritual sin Pantallas", pts: 15 },
                { id: "e3", name: "Dormir a Tiempo", pts: 10 }
            ],
            ejes: {
                standard: { id: "p1", name: "Football Progress: Scouting/Guiones", pts: 30, area: "Content" },
                2: { id: "p2", name: "EDICIÓN INTENSIVA: Football", pts: 40, area: "Fútbol" }, // Martes
                6: { id: "p3", name: "Mantenimiento Z150 (Cadena)", pts: 30, area: "Moto" } // Sábado
            }
        };

        // 2. PERSISTENCIA DE DATOS (localStorage)
        let gioData = JSON.parse(localStorage.getItem('gioData_v3')) || {
            score: 0,
            coins: 0,
            completed: [], // IDs de tareas completadas
            lastDate: new Date().toLocaleDateString(),
            customTasks: DEFAULT_TASKS // Guardamos la definición de tareas
        };

        let isEditMode = false;

        function init() {
            if (gioData.lastDate !== new Date().toLocaleDateString()) resetDaily();
            renderTasks();
            updateUI();
        }

        // 3. RENDERIZADO DE TAREAS (Modo Normal y Edición)
        function renderTasks() {
            const root = document.getElementById('tasks-root');
            root.innerHTML = "";
            const currentTasks = gioData.customTasks;

            const blocks = [
                { id: 'morning-tasks', title: '🌞 Mañana (5:00AM - 7:30AM)', data: currentTasks.morning },
                { id: 'afternoon-tasks', title: '🌤 Tarde (BAMX/Regreso)', data: currentTasks.afternoon },
                { id: 'evening-tasks', title: '🌙 Noche (Cierre)', data: currentTasks.evening }
            ];

            // Renderizar Bloques Principales
            blocks.forEach(b => {
                const section = document.createElement('div');
                section.className = "mb-6";
                section.innerHTML = `<h3 class="text-[10px] text-muted mb-3 uppercase tracking-[0.2em] font-medium">${b.title}</h3>`;
                b.data.forEach(task => section.appendChild(createTaskCard(task)));
                root.appendChild(section);
            });

            // Renderizar Ejes Especiales del Día
            const pilarSection = document.createElement('div');
            pilarSection.className = "border-t border-zinc-800 pt-6";
            pilarSection.innerHTML = `<h3 class="text-[10px] text-accent mb-3 uppercase tracking-[0.2em] font-medium">🎯 Pilar del Día</h3>`;
            const dailyTask = currentTasks.ejes[WEEKDAY] || currentTasks.ejes.standard;
            pilarSection.appendChild(createTaskCard(dailyTask));
            root.appendChild(pilarSection);
        }

        function createTaskCard(task) {
            const div = document.createElement('div');
            const isDone = gioData.completed.includes(task.id);
            div.className = `card task-item p-4 flex justify-between items-center mb-2 cursor-pointer ${isDone ? 'done' : ''}`;
            
            // Lógica para modo edición vs normal
            if (isEditMode) {
                div.innerHTML = `
                    <input type="text" value="${task.name}" onblur="updateTaskText('${task.id}', this.value)" class="task-input text-sm font-medium">
                    <span class="text-[9px] font-bold text-zinc-600">+${task.pts}</span>
                `;
                div.onclick = null; // Desactivar click en modo edición
            } else {
                div.innerHTML = `
                    <div>
                        ${task.area ? `<p class="text-[8px] text-accent uppercase font-bold tracking-widest">${task.area}</p>` : ''}
                        <p class="text-sm font-medium text-white">${task.name}</p>
                    </div>
                    <span class="text-xs font-bold text-muted">+${task.pts}</span>
                `;
                div.onclick = () => toggleTask(div, task);
            }
            return div;
        }

        // 4. LÓGICA DE JUEGO & PUNTOS
        function toggleTask(element, task) {
            if (gioData.completed.includes(task.id)) return;
            element.classList.add('done');
            gioData.completed.push(task.id);
            gioData.score += task.pts;
            updateUI();
            saveData();
        }

        function addSobrietyCoin() {
            gioData.coins += 1;
            alert("🪙 Sobriedad Registrada (+1)");
            updateUI();
            saveData();
        }

        // 5. LÓGICA DE EDICIÓN
        function toggleEditMode() {
            isEditMode = !isEditMode;
            document.getElementById('btn-edit').innerText = isEditMode ? "💾 GUARDAR CAMBIOS" : "✏️ EDITAR TAREAS";
            document.getElementById('btn-edit').classList.toggle('btn-gold');
            renderTasks(); // Volver a dibujar para cambiar inputs por texto
            if (!isEditMode) saveData();
        }

        function updateTaskText(taskId, newText) {
            const categories = ['morning', 'afternoon', 'evening'];
            let taskFound = false;

            for (let cat of categories) {
                let task = gioData.customTasks[cat].find(t => t.id === taskId);
                if (task) { task.name = newText; taskFound = true; break; }
            }
            if (!taskFound) {
                // Check especiales
                if (gioData.customTasks.ejes[taskId]) gioData.customTasks.ejes[taskId].name = newText;
            }
        }

        // 6. UI Y PERSISTENCIA
        function updateUI() {
            document.getElementById('main-score').innerText = gioData.score;
            document.getElementById('coin-count').innerText = gioData.coins;
            const barPercent = Math.min((gioData.score / 150) * 100, 100);
            document.getElementById('progress-bar').style.width = barPercent + "%";
        }

        function saveData() {
            localStorage.setItem('gioData_v3', JSON.stringify(gioData));
        }

        function resetDaily() {
            gioData.score = 0;
            gioData.completed = [];
            gioData.lastDate = new Date().toLocaleDateString();
            saveData();
        }

        function saveBackup() {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(gioData));
            const node = document.createElement('a');
            node.setAttribute("href", dataStr);
            node.setAttribute("download", "progress_executive_backup.json");
            document.body.appendChild(node); node.click(); node.remove();
        }

        init();
    </script>
</body>
</html>
