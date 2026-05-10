<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PROGRESS | Sistema de Control</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #fcfcfc; color: #1a1a1a; }
        .task-card { transition: all 0.2s ease; border: 1px solid #eee; }
        .task-card:active { transform: scale(0.98); }
        .completed { background-color: #f3f3f3 !important; opacity: 0.6; text-decoration: line-through; pointer-events: none; }
        .progress-bar { transition: width 0.5s ease-in-out; }
    </style>
</head>
<body class="max-w-md mx-auto pb-20">

    <header class="p-8 text-center">
        <h1 class="text-4xl font-light tracking-tighter italic">PROGRESS</h1>
        <p class="text-[10px] uppercase tracking-[0.3em] text-gray-400 mt-2">Home Order = Mental Clarity</p>
    </header>

    <section class="px-6 mb-8">
        <div class="bg-white border border-gray-100 p-6 rounded-3xl shadow-sm">
            <div class="flex justify-between items-end mb-4">
                <div>
                    <p class="text-[10px] uppercase font-bold text-gray-400 tracking-widest">Puntos Totales</p>
                    <h2 id="score" class="text-5xl font-bold">0</h2>
                </div>
                <div class="text-right">
                    <p class="text-[10px] uppercase font-bold text-gray-400 tracking-widest">Rango</p>
                    <p id="rank" class="text-xs font-medium text-gray-800">MANTENIMIENTO</p>
                </div>
            </div>
            <div class="w-full bg-gray-100 h-1.5 rounded-full overflow-hidden">
                <div id="bar" class="progress-bar bg-black h-full w-0"></div>
            </div>
            <p class="text-[9px] text-gray-400 mt-2 text-center">META DIARIA: 150 PUNTOS</p>
        </div>
    </section>

    <main class="px-6 space-y-8">
        
        <div>
            <h3 class="text-[10px] font-bold text-gray-300 uppercase tracking-[0.2em] mb-4">🌞 Bloque Mañana (50 pts)</h3>
            <div id="morning-tasks" class="space-y-3"></div>
        </div>

        <div>
            <h3 class="text-[10px] font-bold text-gray-300 uppercase tracking-[0.2em] mb-4">🌤 Bloque Tarde (50 pts)</h3>
            <div id="afternoon-tasks" class="space-y-3"></div>
        </div>

        <div>
            <h3 class="text-[10px] font-bold text-gray-300 uppercase tracking-[0.2em] mb-4">🌙 Bloque Noche (50 pts)</h3>
            <div id="evening-tasks" class="space-y-3"></div>
        </div>

    </main>

    <button onclick="addCoin()" class="fixed bottom-6 right-6 bg-black text-white w-14 h-14 rounded-full shadow-2xl flex items-center justify-center text-xl hover:scale-110 transition">
        🪙
    </button>

    <script>
        const TASKS = {
            morning: [
                { name: "Hacer la cama", pts: 10 },
                { name: "Rutina Higiene (Care)", pts: 10 },
                { name: "Deep Work (10 min)", pts: 15 },
                { name: "Intención Clara (MIT)", pts: 15 }
            ],
            afternoon: [
                { name: "Contactar 3 Aliados", pts: 20 },
                { name: "Hidratación (1L)", pts: 10 },
                { name: "Check-in Emocional", pts: 10 },
                { name: "Ventilar Espacio", pts: 10 }
            ],
            evening: [
                { name: "Orden 10 min", pts: 10 },
                { name: "Cena Saludable", pts: 10 },
                { name: "Ritual sin Pantallas", pts: 10 },
                { name: "Dormir a Tiempo", pts: 10 },
                { name: "Respiración/Gratitud", pts: 10 }
            ]
        };

        let currentScore = 0;

        function init() {
            renderSection('morning-tasks', TASKS.morning);
            renderSection('afternoon-tasks', TASKS.afternoon);
            renderSection('evening-tasks', TASKS.evening);
        }

        function renderSection(containerId, taskList) {
            const container = document.getElementById(containerId);
            taskList.forEach(task => {
                const card = document.createElement('div');
                card.className = "task-card bg-white p-4 rounded-2xl flex justify-between items-center cursor-pointer";
                card.innerHTML = `
                    <span class="text-sm font-medium">${task.name}</span>
                    <span class="text-[10px] font-bold text-gray-400">+${task.pts}</span>
                `;
                card.onclick = () => completeTask(card, task.pts);
                container.appendChild(card);
            });
        }

        function completeTask(element, pts) {
            element.classList.add('completed');
            currentScore += pts;
            updateUI();
        }

        function addCoin() {
            alert("🪙 Moneda de Sobriedad Registrada (+1)");
            updateUI();
        }

        function updateUI() {
            document.getElementById('score').innerText = currentScore;
            
            // Actualizar Rango
            const rank = document.getElementById('rank');
            if (currentScore >= 140) rank.innerText = "🧍‍♂️👑 ÉPICO";
            else if (currentScore >= 80) rank.innerText = "🧍‍♂️🔥 PROGRESO";
            else rank.innerText = "🧍‍♂️ MANTENIMIENTO";

            // Actualizar Barra
            const percent = Math.min((currentScore / 150) * 100, 100);
            document.getElementById('bar').style.width = percent + "%";
        }

        init();
    </script>
</body>
</html>
