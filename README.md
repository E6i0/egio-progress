<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EGIO | Personal Engine</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Plus+Jakarta+Sans:wght@200;400;800&display=swap" rel="stylesheet">
    <style>
        :root { --emerald: #10b981; --bg: #050505; --card: #0f0f0f; }
        body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: var(--bg); color: #e2e8f0; }
        .mono { font-family: 'JetBrains Mono', monospace; }
        .glass { background: var(--card); border: 1px solid #1f1f1f; border-radius: 12px; }
        .achievement-slot { width: 40px; height: 40px; border: 1px dashed #333; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 1.2rem; filter: grayscale(1); opacity: 0.3; transition: all 0.5s; }
        .achievement-slot.unlocked { filter: grayscale(0); opacity: 1; border: 1px solid var(--emerald); background: rgba(16, 185, 129, 0.1); }
        input[type="checkbox"] { width: 1.2rem; height: 1.2rem; border-radius: 4px; accent-color: var(--emerald); cursor: pointer; }
    </style>
</head>
<body class="max-w-xl mx-auto p-6 pb-40">

    <header class="mb-10 pt-4">
        <h1 class="text-xs font-bold tracking-[0.4em] text-emerald-500 mb-2 mono">SYSTEM OVERVIEW</h1>
        <div class="flex justify-between items-end">
            <h2 id="current-day-name" class="text-4xl font-extrabold tracking-tighter"></h2>
            <div class="text-right">
                <p class="text-[10px] text-slate-500 font-bold uppercase">Vida Futura Ganada (Est.)</p>
                <p class="text-xl font-bold text-emerald-400 mono leading-none">+<span id="longevity-val">0.00</span>d</p>
            </div>
        </div>
    </header>

    <section class="glass p-6 mb-8">
        <p class="text-[10px] text-slate-500 font-bold uppercase mb-3 mono">Proyección Disciplina (10 años)</p>
        <div class="flex items-baseline gap-2">
            <span id="projection-val" class="text-4xl font-black text-white">0.0</span>
            <span class="text-emerald-500 font-bold uppercase text-xs">Días de vida extra</span>
        </div>
        <p class="text-[9px] text-slate-600 mt-2 italic">*Basado en reducción de carga cognitiva y optimización de salud.</p>
    </section>

    <section id="daily-zone-container" class="glass p-6 mb-8 border-l-4 border-emerald-500">
        <div class="flex justify-between items-center mb-6">
            <h3 class="text-sm font-bold uppercase tracking-widest text-emerald-500">Micro-Zona</h3>
            <span class="text-[10px] bg-emerald-500/10 text-emerald-500 px-2 py-1 rounded">15-25 MIN</span>
        </div>
        <div id="tasks-list" class="space-y-4">
            </div>
    </section>

    <section class="glass p-6">
        <h3 class="text-[10px] font-bold text-slate-500 uppercase tracking-widest mb-4 mono">Logros: Racha 100%</h3>
        <div id="achievement-wall" class="flex flex-wrap gap-3">
            </div>
    </section>

    <div class="fixed bottom-6 left-1/2 -translate-x-1/2 w-[90%] max-w-lg glass p-4 shadow-2xl">
        <div class="flex justify-between items-center">
            <div class="text-[10px] mono text-slate-400">STATUS: <span id="sync-status" class="text-emerald-500">READY</span></div>
            <button onclick="commitDay()" id="commit-btn" class="bg-emerald-600 hover:bg-emerald-500 text-black font-black text-[10px] uppercase px-6 py-2 rounded-md transition-all">
                Finalizar Jornada
            </button>
        </div>
    </div>

    <script>
        const ZONES = [
            { day: "Domingo", name: "Orden Mental", tasks: ["Revisar semana", "Preparar ropa L-V", "Checklist EGIO"] },
            { day: "Lunes", name: "Habitación", tasks: ["Organizar ropa", "Limpiar superficies", "Revisar outfit semanal"] },
            { day: "Martes", name: "Cocina", tasks: ["Limpiar estufa", "Refrigerador rápido", "Ordenar despensa"] },
            { day: "Miércoles", name: "Baño", tasks: ["Lavabo", "Espejo", "Sanitización rápida"] },
            { day: "Jueves", name: "Área Común", tasks: ["Barrer/Trapear", "Ordenar mesa/escritorio"] },
            { day: "Viernes", name: "Reset General", tasks: ["Basura", "Ropa sucia", "Revisión visual total"] },
            { day: "Sábado", name: "Meal Prep", tasks: ["Preparación alimentos", "Limpieza profunda ligera"] }
        ];

        let currentDayIdx = new Date().getDay();
        let achievements = JSON.parse(localStorage.getItem('egio_achievements')) || 0;

        function init() {
            const zone = ZONES[currentDayIdx];
            document.getElementById('current-day-name').innerText = zone.day;
            
            const list = document.getElementById('tasks-list');
            list.innerHTML = zone.tasks.map((t, i) => `
                <div class="flex items-center gap-4">
                    <input type="checkbox" id="t-${i}" onchange="calculate()">
                    <label for="t-${i}" class="text-sm font-medium tracking-tight text-slate-300">${t}</label>
                </div>
            `).join('');

            renderWall();
            calculate();
        }

        function calculate() {
            const checks = document.querySelectorAll('input[type="checkbox"]');
            const done = [...checks].filter(c => c.checked).length;
            
            // Ingeniería: Cada tarea otorga un valor de longevidad estimada por reducción de cortisol
            const dayLongevity = (done * 0.012); 
            const projection = (dayLongevity * 365 * 10).toFixed(1);

            document.getElementById('longevity-val').innerText = dayLongevity.toFixed(3);
            document.getElementById('projection-val').innerText = projection;

            // Feedback botón
            const btn = document.getElementById('commit-btn');
            if(done === checks.length) {
                btn.classList.add('ring-2', 'ring-emerald-400');
                btn.innerText = "REGISTRAR LOGRO 🏆";
            } else {
                btn.classList.remove('ring-2', 'ring-emerald-400');
                btn.innerText = "Finalizar Jornada";
            }
        }

        function renderWall() {
            const wall = document.getElementById('achievement-wall');
            wall.innerHTML = '';
            for(let i=0; i<21; i++) { // Espacio para 21 logros (3 semanas)
                const slot = document.createElement('div');
                slot.className = `achievement-slot ${i < achievements ? 'unlocked' : ''}`;
                slot.innerText = '💎';
                wall.appendChild(slot);
            }
        }

        function commitDay() {
            const checks = document.querySelectorAll('input[type="checkbox"]');
            const done = [...checks].filter(c => c.checked).length;

            if(done === checks.length) {
                achievements++;
                localStorage.setItem('egio_achievements', achievements);
                alert("Logro desbloqueado: Mente en orden. ¡Felicidades!");
                renderWall();
            } else {
                alert("Corte realizado. Mañana intenta completar todas las micro-zonas para ganar un logro.");
            }
            
            // Reset visual para el demo, en uso real se guardaría en DB
            checks.forEach(c => c.checked = false);
            calculate();
        }

        init();
    </script>
</body>
</html>
