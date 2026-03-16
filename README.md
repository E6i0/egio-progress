<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EGIO OS | Architecture v8.4.2</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Plus+Jakarta+Sans:wght@200;400;700;800&display=swap" rel="stylesheet">
    <style>
        :root { --emerald: #10b981; --bg: #050505; --card: #0f0f0f; }
        body { 
            font-family: 'Plus Jakarta Sans', sans-serif; 
            background-color: var(--bg); 
            color: #e2e8f0; 
            background-image: radial-gradient(circle at 2px 2px, #1a1a1a 1px, transparent 0);
            background-size: 32px 32px;
            padding-bottom: 220px; /* Espacio para que el scroll supere al footer */
        }
        .mono { font-family: 'JetBrains Mono', monospace; }
        .glass { background: rgba(15, 15, 15, 0.85); border: 1px solid rgba(255,255,255,0.05); border-radius: 16px; backdrop-filter: blur(12px); }
        .active-day { border: 2px solid var(--emerald); box-shadow: 0 0 25px rgba(16, 185, 129, 0.2); }
        
        /* FOOTER INTELIGENTE: Se esconde automáticamente */
        .smart-footer {
            position: fixed;
            bottom: 0;
            left: 0;
            width: 100%;
            z-index: 100;
            transform: translateY(75%); /* Escondido */
            opacity: 0.4;
            transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
            padding: 1.5rem;
            background: linear-gradient(to top, black 70%, transparent);
        }
        .smart-footer:hover {
            transform: translateY(0); /* Aparece al acercar el mouse */
            opacity: 1;
        }

        input[type="checkbox"] { width: 1.2rem; height: 1.2rem; accent-color: var(--emerald); cursor: pointer; }
        input[type="checkbox"]:checked + label { color: #475569; text-decoration: line-through; }
        .achievement-slot { width: 12px; height: 12px; border-radius: 2px; background: #1a1a1a; transition: all 0.5s; }
        .achievement-slot.filled { background: var(--emerald); box-shadow: 0 0 8px var(--emerald); }
    </style>
</head>
<body class="max-w-6xl mx-auto p-4 md:p-8">

    <header class="flex justify-between items-end mb-12">
        <div>
            <h1 class="text-[10px] font-bold tracking-[0.5em] text-emerald-500 mb-2 mono">EGIO OS | CORE V8.4.2</h1>
            <h2 class="text-4xl font-extrabold tracking-tighter italic">Abundance Engine</h2>
        </div>
        <div class="text-right glass p-5 border-t-2 border-emerald-500">
            <p class="text-[9px] text-slate-500 font-bold uppercase mono">Longevidad Acumulada</p>
            <p class="text-3xl font-bold text-emerald-400 mono">+<span id="total-longevity">0.000</span></p>
        </div>
    </header>

    <main class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 mb-12" id="week-grid"></main>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <section class="glass p-8">
            <h3 class="text-[10px] font-bold tracking-widest text-emerald-500 mono mb-6 uppercase">🌅 Ritual de Apertura</h3>
            <div id="ritual-tasks" class="space-y-4"></div>
        </section>
        <section class="glass p-8 border-l-4 border-emerald-500">
            <h3 class="text-[10px] font-bold tracking-widest text-emerald-500 mono mb-6 uppercase">🏠 Micro-Zona: <span id="current-zone-name"></span></h3>
            <div id="micro-tasks" class="space-y-4"></div>
        </section>
        <section class="glass p-8">
            <h3 class="text-[10px] font-bold tracking-widest text-emerald-500 mono mb-6 uppercase">💻 Digital Engine</h3>
            <div id="digital-tasks" class="space-y-4"></div>
        </section>
        <section class="glass p-8">
            <h3 class="text-[10px] font-bold tracking-widest text-emerald-500 mono mb-6 uppercase">💰 Monetización</h3>
            <div id="monetization-tasks" class="space-y-4"></div>
        </section>
    </div>

    <footer class="smart-footer">
        <div class="max-w-6xl mx-auto glass p-6 shadow-2xl flex flex-col md:flex-row justify-between items-center gap-6 border-t border-emerald-500/20">
            <div>
                <p class="text-[9px] text-slate-500 uppercase font-bold mono mb-2">Consistencia (30D)</p>
                <div id="achievement-grid" class="flex flex-wrap gap-1"></div>
            </div>
            <div class="flex gap-4">
                <button onclick="exportHistory()" class="px-6 py-4 rounded-lg border border-slate-700 text-slate-400 text-[10px] font-bold mono hover:bg-white/5 transition-all">RESPALDO .TXT</button>
                <button onclick="executeCut()" class="bg-emerald-600 hover:bg-emerald-400 text-black font-black text-[10px] uppercase px-12 py-4 rounded-lg transition-all tracking-widest shadow-lg shadow-emerald-900/30">EJECUTAR CORTE</button>
            </div>
        </div>
    </footer>

    <script>
        const PLAN = {
            ritual: ["Tender cama", "Utensilios lavados", "Ventilación", "Desayuno funcional"],
            digital: ["Programar contenido", "Optimización de ideas", "Limpieza de Inbox"],
            monetization: ["Batching producción", "Revisión analíticas", "Brainstorming"],
            weekly: [
                { id: 0, d: "Domingo", z: "Orden Mental", t: ["Revisar semana", "Preparar ropa L-V", "Checklist EGIO"] },
                { id: 1, d: "Lunes", z: "Habitación", t: ["Organizar ropa", "Limpiar superficies", "Revisar outfit"] },
                { id: 2, d: "Martes", z: "Cocina", t: ["Limpiar estufa", "Refrigerador", "Ordenar despensa"] },
                { id: 3, d: "Miércoles", z: "Baño", t: ["Lavabo", "Espejo", "Sanitización rápida"] },
                { id: 4, d: "Jueves", z: "Área Común", t: ["Barrer/Trapear", "Ordenar mesa"] },
                { id: 5, d: "Viernes", z: "Reset General", t: ["Basura", "Ropa sucia", "Revisión visual"] },
                { id: 6, d: "Sábado", z: "Producción", t: ["Meal prep simple", "Limpieza profunda ligera"] }
            ]
        };

        let state = JSON.parse(localStorage.getItem('egio_v8_state')) || { checks: {}, history: 0, lastCut: "" };
        const today = new Date().getDay();

        function init() {
            const grid = document.getElementById('week-grid');
            grid.innerHTML = PLAN.weekly.map(item => `
                <div class="glass p-6 ${item.id === today ? 'active-day' : 'opacity-60'}">
                    <div class="flex justify-between items-start mb-4">
                        <h3 class="text-[10px] font-bold uppercase tracking-widest text-emerald-500 mono">${item.d}</h3>
                        ${item.id === today ? '<span class="text-[8px] bg-emerald-500/10 text-emerald-500 px-2 py-0.5 rounded mono">HOY</span>' : ''}
                    </div>
                    <h4 class="text-xl font-bold mb-6 tracking-tight">${item.z}</h4>
                    <div class="space-y-4">
                        ${item.t.map((task, i) => {
                            const uid = `weekly-${item.id}-${i}`;
                            return `
                                <div class="flex items-center gap-3 group">
                                    <input type="checkbox" id="${uid}" onchange="toggle('${uid}')" ${state.checks[uid] ? 'checked' : ''}>
                                    <label for="${uid}" class="text-xs font-medium text-slate-400 group-hover:text-white cursor-pointer">${task}</label>
                                </div>`;
                        }).join('')}
                    </div>
                </div>
            `).join('');

            renderTasks('ritual-tasks', 'ritual', PLAN.ritual);
            renderTasks('digital-tasks', 'digital', PLAN.digital);
            renderTasks('monetization-tasks', 'monet', PLAN.monetization);
            
            const todayPlan = PLAN.weekly[today];
            document.getElementById('current-zone-name').innerText = todayPlan.z;
            renderTasks('micro-tasks', 'micro', todayPlan.t);
            
            renderAchievements();
            calculate();
        }

        function renderTasks(containerId, prefix, tasks) {
            document.getElementById(containerId).innerHTML = tasks.map((t, i) => {
                const uid = `${prefix}-${i}`;
                return `<div class="flex items-center gap-4"><input type="checkbox" id="${uid}" onchange="toggle('${uid}')" ${state.checks[uid] ? 'checked' : ''}><label for="${uid}" class="text-sm text-slate-300 cursor-pointer">${t}</label></div>`;
            }).join('');
        }

        function toggle(uid) { state.checks[uid] = document.getElementById(uid).checked; save(); calculate(); }
        function calculate() { 
            const total = Object.values(state.checks).filter(v => v).length;
            document.getElementById('total-longevity').innerText = (total * 0.012).toFixed(3);
        }
        function renderAchievements() {
            const container = document.getElementById('achievement-grid');
            container.innerHTML = Array.from({length: 30}).map((_, i) => `<div class="achievement-slot ${i < state.history ? 'filled' : ''}"></div>`).join('');
        }
        function executeCut() {
            const todayStr = new Date().toISOString().split('T')[0];
            if(state.lastCut === todayStr) return alert("Corte ya realizado.");
            state.history++; state.lastCut = todayStr; save(); renderAchievements();
            alert("SISTEMA SINCRONIZADO.");
        }
        function exportHistory() {
            const data = `EGIO LOG\nLongevidad: ${document.getElementById('total-longevity').innerText}\n\n` + Object.entries(state.checks).filter(([k,v]) => v).map(([k,v]) => `[OK] ${k}`).join('\n');
            const blob = new Blob([data], {type: 'text/plain'});
            const a = document.createElement('a');
            a.href = URL.createObjectURL(blob);
            a.download = `EGIO_Backup.txt`;
            a.click();
        }
        function save() { localStorage.setItem('egio_v8_state', JSON.stringify(state)); }
        init();
    </script>
</body>
</html>
