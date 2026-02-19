<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EGIO OS | Personal Architecture</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@200;400;700;800&display=swap" rel="stylesheet">
    <style>
        :root { --bg: #0a0f1a; --card: #141b2d; --accent: #10b981; --text-dim: #94a3b8; }
        body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: var(--bg); color: #f8fafc; overflow-x: hidden; }
        .glass { background: rgba(20, 27, 45, 0.8); backdrop-filter: blur(20px); border: 1px solid rgba(255, 255, 255, 0.05); }
        .neo-card { border-radius: 28px; transition: transform 0.3s ease; }
        .neo-card:hover { transform: translateY(-4px); }
        .check-pill { transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1); }
        input[type="checkbox"]:checked + label { color: var(--text-dim); text-decoration: line-through; opacity: 0.6; }
        .mood-active { filter: drop-shadow(0 0 8px var(--accent)); transform: scale(1.2); grayscale: 0; }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-thumb { background: #334155; border-radius: 10px; }
    </style>
</head>
<body class="max-w-2xl mx-auto p-6 pb-48">

    <header class="flex justify-between items-center mb-12">
        <div>
            <h1 class="text-xs font-black tracking-[0.5em] text-emerald-500 uppercase mb-1">Architecture v4.0</h1>
            <p id="header-date" class="text-2xl font-light tracking-tight text-white"></p>
        </div>
        <div class="h-10 w-10 glass rounded-full flex items-center justify-center border border-emerald-500/20">
            <div class="h-2 w-2 bg-emerald-500 rounded-full animate-pulse"></div>
        </div>
    </header>

    <section class="grid grid-cols-2 gap-4 mb-10">
        <div class="glass p-6 neo-card">
            <span class="text-[10px] font-bold text-slate-500 uppercase tracking-widest">Longevidad Acumulada</span>
            <div class="text-3xl font-extrabold mt-1 text-emerald-400 leading-none">+<span id="longevity">0.0</span></div>
        </div>
        <div class="glass p-6 neo-card">
            <span class="text-[10px] font-bold text-slate-500 uppercase tracking-widest">Minimalismo Mental</span>
            <div class="text-3xl font-extrabold mt-1 text-blue-400 leading-none"><span id="clarity">0</span>%</div>
        </div>
    </section>

    <div id="dynamic-engine" class="space-y-6">
        </div>

    <div class="fixed bottom-8 left-1/2 -translate-x-1/2 w-[90%] max-w-lg glass p-6 neo-card border-t border-emerald-500/30 shadow-2xl">
        <div class="flex flex-col gap-6">
            <div class="flex items-center justify-between">
                <div class="flex gap-4" id="mood-selector">
                    <button onclick="setMood('sad', this)" class="text-2xl opacity-40 grayscale hover:grayscale-0 hover:opacity-100 transition-all">😢</button>
                    <button onclick="setMood('neutral', this)" class="text-2xl opacity-40 grayscale hover:grayscale-0 hover:opacity-100 transition-all">😐</button>
                    <button onclick="setMood('happy', this)" class="text-2xl opacity-40 grayscale hover:grayscale-0 hover:opacity-100 transition-all">😊</button>
                </div>
                <button onclick="handleExport()" class="bg-white text-black text-[10px] font-black px-6 py-3 rounded-full uppercase tracking-widest hover:bg-emerald-500 hover:text-white transition-all">
                    Ejecutar Corte
                </button>
            </div>
            <textarea id="daily-note" rows="1" placeholder="Resumen estratégico del día..." 
                class="bg-transparent text-sm border-b border-white/10 pb-2 focus:border-emerald-500 outline-none transition-all resize-none"></textarea>
        </div>
    </div>

    <script>
        const PLAN = [
            { id: 'morning', icon: '🌅', title: 'Ritual de Apertura', tasks: ['Tender cama (Orden visual)', 'Utensilios lavados', 'Ventilación habitación', 'Desayuno funcional (Ritual)'] },
            { id: 'work', icon: '🏢', title: 'Ejecución BAMX', tasks: ['3 Tareas clave BAMX', 'Seguimiento institucional', 'Logística de eventos'] },
            { id: 'zone', icon: '🏠', title: 'Micro-Zona', dynamic: true },
            { id: 'digital', icon: '💻', title: 'Monetización & Contenido', tasks: ['Programar publicaciones', 'Optimización de ideas', 'Batching digital'] }
        ];

        const WEEK_ZONES = [
            { n: "Mente & Planificación", t: ["Revisar semana", "Preparar ropa L-V", "Checklist EGIO"] }, // Dom
            { n: "Habitación (Orden)", t: ["Organizar ropa", "Limpiar superficies", "Vaciado de burós"] }, // Lun
            { n: "Cocina (Reset)", t: ["Limpiar estufa", "Refrigerador rápido", "Orden despensa"] }, // Mar
            { n: "Baño (Ritual)", t: ["Espejos y lavabo", "Higiene profunda", "Sanitización"] }, // Mie
            { n: "Área Común (Flujo)", t: ["Barrer/Trapear", "Ordenar escritorio", "Orden visual total"] }, // Jue
            { n: "Reset General", t: ["Gestionar basura", "Ropa sucia", "Revisión visual final"] }, // Vie
            { n: "Meal Prep & Producción", t: ["Batch cooking simple", "Limpieza profunda ligera", "Compras personales"] } // Sab
        ];

        let currentKey = "";
        let mood = null;

        function init() {
            const today = new Date();
            currentKey = "egio_" + today.toISOString().split('T')[0];
            document.getElementById('header-date').innerText = today.toLocaleDateString('es-MX', { weekday: 'short', day: 'numeric', month: 'long' });
            
            render(today.getDay());
            load();
        }

        function render(dayIdx) {
            const container = document.getElementById('dynamic-engine');
            container.innerHTML = PLAN.map(block => {
                const title = block.dynamic ? block.icon + " " + WEEK_ZONES[dayIdx].n : block.icon + " " + block.title;
                const tasks = block.dynamic ? WEEK_ZONES[dayIdx].t : block.tasks;

                return `
                    <div class="glass p-8 neo-card">
                        <h2 class="text-xs font-black uppercase tracking-[0.2em] text-emerald-500/60 mb-6">${title}</h2>
                        <div class="space-y-5">
                            ${tasks.map(task => {
                                const uid = btoa(title + task).slice(0, 8);
                                return `
                                    <div class="flex items-center gap-4 group">
                                        <div class="relative flex items-center justify-center">
                                            <input type="checkbox" id="${uid}" onchange="save()" 
                                                class="peer appearance-none h-5 w-5 border border-white/20 rounded-full checked:bg-emerald-500 checked:border-transparent transition-all cursor-pointer">
                                            <div class="absolute pointer-events-none hidden peer-checked:block text-black text-[10px] font-bold">✓</div>
                                        </div>
                                        <label for="${uid}" class="text-sm font-medium text-slate-300 cursor-pointer group-hover:text-white transition-colors">${task}</label>
                                    </div>`;
                            }).join('')}
                        </div>
                    </div>`;
            }).join('');
        }

        function save() {
            const checks = {};
            document.querySelectorAll('input[type="checkbox"]').forEach(c => checks[c.id] = c.checked);
            localStorage.setItem(currentKey, JSON.stringify({ 
                checks, 
                mood, 
                note: document.getElementById('daily-note').value 
            }));
            updateStats();
        }

        function load() {
            const data = JSON.parse(localStorage.getItem(currentKey)) || { checks: {}, mood: null, note: "" };
            document.querySelectorAll('input[type="checkbox"]').forEach(c => c.checked = data.checks[c.id] || false);
            document.getElementById('daily-note').value = data.note || "";
            if(data.mood) setMood(data.mood, null);
            updateStats();
        }

        function updateStats() {
            const cbs = document.querySelectorAll('input[type="checkbox"]');
            const done = [...cbs].filter(c => c.checked).length;
            const pct = cbs.length ? Math.round((done / cbs.length) * 100) : 0;
            document.getElementById('clarity').innerText = pct;
            document.getElementById('longevity').innerText = (done * 0.05).toFixed(1);
        }

        function setMood(m, el) {
            mood = m;
            document.querySelectorAll('#mood-selector button').forEach(b => {
                b.style.filter = "grayscale(1)";
                b.style.opacity = "0.4";
                b.style.transform = "scale(1)";
            });
            const activeBtn = el || [...document.querySelectorAll('#mood-selector button')][m === 'sad' ? 0 : m === 'neutral' ? 1 : 2];
            activeBtn.style.filter = "grayscale(0)";
            activeBtn.style.opacity = "1";
            activeBtn.style.transform = "scale(1.2)";
            save();
        }

        function handleExport() {
            if(!mood) return alert("Define tu estado emocional para el corte.");
            const date = currentKey.split('_')[1];
            const data = `Fecha: ${date}\nLongevidad: ${document.getElementById('longevity').innerText}\nClaridad: ${document.getElementById('clarity').innerText}%\nNota: ${document.getElementById('daily-note').value}`;
            const blob = new Blob([data], { type: 'text/plain' });
            const a = document.createElement('a');
            a.href = URL.createObjectURL(blob);
            a.download = `EGIO_Corte_${date}.txt`;
            a.click();
            alert("✅ Corte local generado.");
        }

        init();
    </script>
</body>
</html>
