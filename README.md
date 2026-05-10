<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PROGRESS | Tactical Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;500;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'JetBrains Mono', monospace; background-color: #0a0a0a; color: #e5e5e5; }
        .card { background-color: #141414; border: 1px solid #262626; border-radius: 16px; }
        .task-done { border-color: #3b82f6; background-color: #1e293b; opacity: 0.6; text-decoration: line-through; }
        .progress-fill { transition: width 0.8s cubic-bezier(0.4, 0, 0.2, 1); }
        .day-box { width: 12px; height: 12px; border-radius: 2px; background-color: #262626; }
        .day-active { background-color: #3b82f6; box-shadow: 0 0 8px #3b82f6; }
    </style>
</head>
<body class="max-w-md mx-auto p-6 pb-24">

    <header class="mb-8">
        <div class="flex justify-between items-center mb-2">
            <h1 class="text-2xl font-extrabold tracking-tighter italic text-white">PROGRESS</h1>
            <span id="current-day-label" class="text-[10px] bg-blue-600 text-white px-2 py-1 rounded">MODO: ESTÁNDAR</span>
        </div>
        <div class="w-full bg-zinc-900 h-1 rounded-full overflow-hidden">
            <div id="year-progress" class="bg-zinc-500 h-full w-0 progress-fill"></div>
        </div>
        <div class="flex justify-between mt-1">
            <p class="text-[8px] text-zinc-500 uppercase">Progreso Año 2026</p>
            <p id="days-left" class="text-[8px] text-zinc-500 uppercase">-- días restantes</p>
        </div>
    </header>

    <section class="card p-4 mb-6">
        <p class="text-[9px] text-zinc-500 uppercase mb-3">Rendimiento Semanal</p>
        <div class="flex justify-between px-2" id="week-calendar">
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="d0"></div><span class="text-[7px]">L</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="d1"></div><span class="text-[7px]">M</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="d2"></div><span class="text-[7px]">M</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="d3"></div><span class="text-[7px]">J</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="d4"></div><span class="text-[7px]">V</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="d5"></div><span class="text-[7px]">S</span></div>
            <div class="flex flex-col items-center gap-1"><div class="day-box" id="d6"></div><span class="text-[7px]">D</span></div>
        </div>
    </section>

    <section class="card p-6 mb-8 border-l-4 border-l-blue-600">
        <div class="flex justify-between items-end">
            <div>
                <p class="text-[10px] text-zinc-500 uppercase font-bold tracking-widest">Puntos Hoy</p>
                <h2 id="score" class="text-6xl font-black text-white">0</h2>
            </div>
            <div class="text-right">
                <p class="text-[10px] text-zinc-500 uppercase font-bold tracking-widest">Status</p>
                <p id="rank" class="text-xs font-bold text-blue-400">MANTENIMIENTO</p>
            </div>
        </div>
    </section>

    <main id="tasks-container" class="space-y-4">
        </main>

    <button onclick="alert('🪙 Sobriedad Confirmada')" class="fixed bottom-6 right-6 bg-white text-black w-14 h-14 rounded-full shadow-2xl flex items-center justify-center hover:scale-110 transition">
        <span class="text-xl font-bold">+1</span>
    </button>

    <script>
        const DATE = new Date();
        const WEEKDAY = DATE.getDay(); // 0=D, 1=L... 6=S

        const MISSION_DATA = {
            standard: [ // Lunes a Viernes
                { name: "Deep Work (10 min)", pts: 15, area: "Mental" },
                { name: "Intención MIT", pts: 15, area: "Mental" },
                { name: "Aliados (3 contactos)", pts: 20, area: "Trabajo" },
                { name: "Hidratación (1L)", pts: 10, area: "Salud" },
                { name: "Orden 10 min", pts: 10, area: "Salud" }
            ],
            saturday: [ // SÁBADO: MANTENIMIENTO Z150
                { name: "Aseo Profundo Casa", pts: 30, area: "Orden" },
                { name: "Lavado Detallado Z150", pts: 20, area: "Moto" },
                { name: "Revisión Cadena/Aceite Z150", pts: 20, area: "Moto" },
                { name: "Cambio de Sábanas", pts: 10, area: "Orden" },
                { name: "Cita con Serrat (Foco Total)", pts: 40, area: "Pareja" }
            ],
            sunday: [ // DOMINGO: ESTRATEGIA
                { name: "Planificación Semanal PROGRESS", pts: 40, area: "Estrategia" },
                { name: "Carga Nula (Inbox 0)", pts: 20, area: "Mental" },
                { name: "Lectura / Skill-up", pts: 30, area: "Mental" },
                { name: "Prep Comidas Semanal", pts: 30, area: "Salud" },
                { name: "Cierre Financiero Semanal", pts: 30, area: "Finanzas" }
            ]
        };

        let currentScore = 0;

        function init() {
            updateTimeMetrics();
            loadDailyPillar();
            highlightCurrentDay();
        }

        function updateTimeMetrics() {
            const now = new Date();
            const start = new Date(now.getFullYear(), 0, 0);
            const diff = now - start;
            const oneDay = 1000 * 60 * 60 * 24;
            const dayOfYear = Math.floor(diff / oneDay);
            const percentYear = (dayOfYear / 365) * 100;
            
            document.getElementById('year-progress').style.width = percentYear + "%";
            document.getElementById('days-left').innerText = (365 - dayOfYear) + " DÍAS RESTANTES";
        }

        function highlightCurrentDay() {
            // Ajustar JS (0=Dom) a tu visual (L=0, D=6)
            let visualDay = WEEKDAY === 0 ? 6 : WEEKDAY - 1;
            document.getElementById('d' + visualDay).classList.add('day-active');
        }

        function loadDailyPillar() {
            const container = document.getElementById('tasks-container');
            const label = document.getElementById('current-day-label');
            let activeTasks = MISSION_DATA.standard;

            if (WEEKDAY === 6) {
                activeTasks = MISSION_DATA.saturday;
                label.innerText = "MODO: MANTENIMIENTO & Z150";
                label.classList.replace('bg-blue-600', 'bg-orange-600');
            } else if (WEEKDAY === 0) {
                activeTasks = MISSION_DATA.sunday;
                label.innerText = "MODO: ESTRATEGIA & RESET";
                label.classList.replace('bg-blue-600', 'bg-purple-600');
            }

            activeTasks.forEach(task => {
                const div = document.createElement('div');
                div.className = "card p-5 flex justify-between items-center cursor-pointer transition active:scale-95";
                div.innerHTML = `
                    <div>
                        <p class="text-[8px] text-zinc-500 uppercase tracking-widest font-bold">${task.area}</p>
                        <p class="text-sm font-medium text-white">${task.name}</p>
                    </div>
                    <span class="text-xs font-black text-zinc-600">+${task.pts}</span>
                `;
                div.onclick = () => {
                    if(!div.classList.contains('task-done')) {
                        div.classList.add('task-done');
                        currentScore += task.pts;
                        updateScore();
                    }
                };
                container.appendChild(div);
            });
        }

        function updateScore() {
            document.getElementById('score').innerText = currentScore;
            const rank = document.getElementById('rank');
            if (currentScore >= 120) rank.innerText = "HOMBRE ÉPICO";
            else if (currentScore >= 70) rank.innerText = "EN PROGRESO";
            else rank.innerText = "MANTENIMIENTO";
        }

        init();
    </script>
</body>
</html>
