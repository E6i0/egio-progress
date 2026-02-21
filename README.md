<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EGIO OS | Architecture v8.0</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Plus+Jakarta+Sans:wght@200;400;700;800&display=swap" rel="stylesheet">
    <style>
        :root { --emerald: #10b981; --bg: #050505; --card: #0f0f0f; }
        body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: var(--bg); color: #e2e8f0; scroll-behavior: smooth; }
        .mono { font-family: 'JetBrains Mono', monospace; }
        .glass { background: var(--card); border: 1px solid #1a1a1a; border-radius: 16px; transition: all 0.3s ease; }
        .active-day { border: 1px solid var(--emerald); box-shadow: 0 0 20px rgba(16, 185, 129, 0.1); }
        input[type="checkbox"] { width: 1.2rem; height: 1.2rem; accent-color: var(--emerald); cursor: pointer; border-radius: 4px; }
        input[type="checkbox"]:checked + label { color: #475569; text-decoration: line-through; }
        .achievement-slot { width: 12px; height: 12px; border-radius: 2px; background: #1a1a1a; transition: all 0.5s; }
        .achievement-slot.filled { background: var(--emerald); box-shadow: 0 0 8px var(--emerald); }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-thumb { background: #1a1a1a; border-radius: 10px; }
    </style>
</head>
<body class="max-w-4xl mx-auto p-4 md:p-8 pb-40">

    <header class="flex justify-between items-end mb-12">
        <div>
            <h1 class="text-[10px] font-bold tracking-[0.5em] text-emerald-500 mb-2 mono">SYSTEM ARCHITECTURE V8.0</h1>
            <h2 class="text-4xl font-extrabold tracking-tighter">Planificación Semanal</h2>
        </div>
        <div class="text-right glass p-4">
            <p class="text-[9px] text-slate-500 font-bold uppercase mono">Longevidad Total Semanal</p>
            <p class="text-2xl font-bold text-emerald-400 mono">+<span id="total-longevity">0.000</span></p>
        </div>
    </header>

    <main class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6" id="week-grid">
        </main>

    <footer class="fixed bottom-6 left-1/2 -translate-x-1/2 w-[92%] max-w-4xl glass p-6 shadow-2xl flex flex-col md:flex-row justify-between items-center gap-6 border-t border-emerald-500/20">
        <div class="flex flex-col gap-2">
            <p class="text-[9px] text-slate-500 uppercase font-bold mono">Consistencia de Hábitos</p>
            <div id="achievement-grid" class="flex gap-1">
                </div>
        </div>
        
        <div class="flex items-center gap-4 w-full md:w-auto">
            <button onclick="executeCut()" class="w-full md:w-auto bg-emerald-600 hover:bg-emerald-400 text-black font-black text-[10px] uppercase px-10 py-4 rounded-sm transition-all tracking-widest">
                Ejecutar Corte y Archivar
            </button>
        </div>
    </footer>

    <script>
        const WEEK_PLAN = [
            { id: 0, day: "Domingo", zone: "Orden Mental", tasks: ["Revisar semana", "Preparar ropa L-V", "Checklist EGIO"] },
            { id: 1, day: "Lunes", zone: "Habitación", tasks: ["Organizar ropa", "Limpiar superficies", "Revisar outfit semanal"] },
            { id: 2, day: "Martes", zone: "Cocina", tasks: ["Limpiar estufa", "Refrigerador rápido", "Ordenar despensa"] },
            { id: 3, day: "Miércoles", zone: "Baño", tasks: ["Lavabo", "Espejo", "Sanitización rápida"] },
            { id: 4, day: "Jueves", zone: "Área Común", tasks: ["Barrer/Trapear", "Ordenar mesa/escritorio"] },
            { id: 5, day: "Viernes", zone: "Reset General", tasks: ["Basura", "Ropa sucia", "Revisión visual total"] },
            { id: 6, day: "Sábado", zone: "Producción", tasks: ["Meal prep simple", "Limpieza profunda ligera"] }
        ];

        let state = JSON.parse(localStorage.getItem('egio_v8_state')) || {
            checks: {}, // Formato: "diaID-tareaID": true/false
            history: 0
        };

        function init() {
            const grid = document.getElementById('week-grid');
            const today = new Date().getDay();

            grid.innerHTML = WEEK_PLAN.map(item => `
                <div class="glass p-6 ${item.id === today ? 'active-day' : 'opacity-60'}">
                    <div class="flex justify-between items-start mb-4">
                        <h3 class="text-[10px] font-bold uppercase tracking-widest text-emerald-500 mono">${item.day}</h3>
                        ${item.id === today ? '<span class="text-[8px] bg-emerald-500/10 text-emerald-500 px-2 py-0.5 rounded mono">HOY</span>' : ''}
                    </div>
                    <h4 class="text-xl font-bold mb-6 tracking-tight">${item.zone}</h4>
                    <div class="space-y-4">
                        ${item.tasks.map((task, i) => {
                            const uid = `${item.id}-${i}`;
                            const checked = state.checks[uid] ? 'checked' : '';
                            return `
                                <div class="flex items-center gap-3 group">
                                    <input type="checkbox" id="${uid}" onchange="toggleTask('${uid}')" ${checked}>
                                    <label for="${uid}" class="text-xs font-medium text-slate-400 group-hover:text-white transition-colors cursor-pointer">${task}</label>
                                </div>
                            `;
                        }).join('')}
                    </div>
                </div>
            `).join('');

            renderAchievements();
            calculateLongevity();
        }

        function toggleTask(uid) {
            const cb = document.getElementById(uid);
            state.checks[uid] = cb.checked;
            save();
            calculateLongevity();
        }

        function calculateLongevity() {
            const totalChecks = Object.values(state.checks).filter(v => v === true).length;
            const longevityVal = (totalChecks * 0.012).toFixed(3);
            document.getElementById('total-longevity').innerText = longevityVal;
        }

        function renderAchievements() {
            const container = document.getElementById('achievement-grid');
            container.innerHTML = '';
            for(let i = 0; i < 28; i++) {
                const dot = document.createElement('div');
                dot.className = `achievement-slot ${i < state.history ? 'filled' : ''}`;
                container.appendChild(dot);
            }
        }

        function executeCut() {
            const today = new Date().getDay();
            const todayTasks = WEEK_PLAN[today].tasks.length;
            
            // Verificar si el día actual está completo
            let completedToday = 0;
            WEEK_PLAN[today].tasks.forEach((_, i) => {
                if(state.checks[`${today}-${i}`]) completedToday++;
            });

            if(completedToday === todayTasks) {
                state.history++;
                alert("CORTE EXITOSO: Se ha registrado un incremento en tu claridad mental y longevidad.");
            } else {
                alert("AVISO: El día actual no está completo al 100%. Se archiva progreso parcial.");
            }
            
            save();
            renderAchievements();
        }

        function save() {
            localStorage.setItem('egio_v8_state', JSON.stringify(state));
        }

        init();
    </script>
</body>
</html>
