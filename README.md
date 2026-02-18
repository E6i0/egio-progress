<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EGIO OS | Excel Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;600;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: #0f172a; color: #f1f5f9; }
        .glass-card { background: rgba(30, 41, 59, 0.7); backdrop-filter: blur(12px); border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 20px; }
        .accent-border { border-left: 4px solid #38bdf8; }
        input[type="checkbox"]:checked + label { color: #64748b; text-decoration: line-through; }
        .mood-btn { filter: grayscale(1); transition: all 0.2s; opacity: 0.5; font-size: 1.5rem; }
        .mood-btn.active { filter: grayscale(0); opacity: 1; transform: scale(1.3); }
        .export-btn { background: linear-gradient(90deg, #10b981, #3b82f6); color: white; font-weight: 800; }
    </style>
</head>
<body class="max-w-xl mx-auto p-4 pb-48">

    <header class="py-8 text-center">
        <h1 class="text-3xl font-extrabold tracking-tighter bg-gradient-to-r from-emerald-400 to-blue-400 bg-clip-text text-transparent text-center">EGIO PROGRESS</h1>
        <div class="flex justify-center items-center gap-2 mt-2">
            <input type="date" id="date-picker" class="bg-transparent text-xs text-slate-400 font-bold uppercase tracking-widest cursor-pointer outline-none text-center">
        </div>
    </header>

    <div class="grid grid-cols-2 gap-4 mb-6 text-center">
        <div class="glass-card p-4 border-b-2 border-emerald-500/30">
            <p class="text-[10px] text-slate-400 font-bold uppercase mb-1 italic">Longevidad</p>
            <p class="text-2xl font-bold text-emerald-400">+<span id="longevity-score">0.0</span></p>
        </div>
        <div class="glass-card p-4 border-b-2 border-blue-500/30">
            <p class="text-[10px] text-slate-400 font-bold uppercase mb-1 italic">Claridad Mental</p>
            <p id="clarity-percent" class="text-2xl font-bold text-blue-400">0%</p>
        </div>
    </div>

    <main id="app-engine" class="space-y-4"></main>

    <div class="fixed bottom-6 left-1/2 -translate-x-1/2 w-[92%] max-w-lg glass-card p-5 border-t border-emerald-500/30 shadow-2xl">
        <div class="flex flex-col gap-4">
            <div class="flex items-center gap-4">
                <div class="flex gap-3 border-r border-slate-700 pr-4">
                    <button onclick="updateMood('sad')" id="m-sad" class="mood-btn">😢</button>
                    <button onclick="updateMood('neutral')" id="m-neutral" class="mood-btn">😐</button>
                    <button onclick="updateMood('happy')" id="m-happy" class="mood-btn">😊</button>
                </div>
                <textarea id="daily-note" rows="1" placeholder="Reseña del día..." 
                    class="flex-1 bg-transparent text-sm outline-none resize-none py-1"></textarea>
            </div>
            
            <button onclick="exportToExcel()" class="export-btn w-full py-3 rounded-xl text-[10px] uppercase tracking-[0.2em] shadow-lg shadow-emerald-500/20 active:scale-95 transition-all">
                Hacer Corte (Descargar Excel/CSV) 📥
            </button>
        </div>
    </div>

    <script>
        const MASTER_PLAN = [
            { title: "🌅 Ritual Mañana", tasks: ["Tender cama", "Lavar utensilios desayuno", "Ventilación", "Desayuno funcional"] },
            { title: "🏢 Enfoque BAMX", tasks: ["3 Tareas clave", "Contacto aliados", "Seguimiento acopio"] },
            { title: "🏠 Zona del Día", dynamic: true },
            { title: "💻 Digital/Monetización", tasks: ["Batching ideas", "Programar contenido", "Limpieza Inbox"] }
        ];

        const ZONES = [
            { name: "Mente: Planificación", tasks: ["Revisar semana", "Outfit L-V", "Checklist EGIO"] }, // Dom
            { name: "Habitación: Orden", tasks: ["Organizar ropa", "Limpiar superficies", "Vaciado de burós"] }, // Lun
            { name: "Cocina: Reset", tasks: ["Limpiar estufa", "Refrigerador rápido", "Despensa"] }, // Mar
            { name: "Baño: Ritual", tasks: ["Espejos y lavabo", "Higiene profunda", "Sanitización"] }, // Mie
            { name: "Comunes: Flujo", tasks: ["Barrer/Trapear", "Limpiar Escritorio", "Orden visual"] }, // Jue
            { name: "Basics: Vaciado", tasks: ["Gestionar basura", "Ropa sucia", "Reset total"] }, // Vie
            { name: "Producción: Batch", tasks: ["Grabación contenido", "Meal Prep", "Compras"] } // Sab
        ];

        let currentKey = "";
        window.currentMood = null;

        function init() {
            const picker = document.getElementById('date-picker');
            picker.valueAsDate = new Date();
            picker.addEventListener('change', loadDay);
            document.getElementById('daily-note').addEventListener('input', saveToStorage);
            loadDay();
        }

        function loadDay() {
            const dateStr = document.getElementById('date-picker').value;
            const date = new Date(dateStr + "T00:00:00");
            currentKey = "egio_" + dateStr;
            renderUI(date.getDay());
            
            const data = JSON.parse(localStorage.getItem(currentKey)) || { checks: {}, mood: null, note: "" };
            document.querySelectorAll('input[type="checkbox"]').forEach(cb => cb.checked = data.checks[cb.id] || false);
            document.getElementById('daily-note').value = data.note || "";
            window.currentMood = data.mood;
            updateMoodUI(data.mood);
            updateStats();
        }

        function renderUI(dayNum) {
            const container = document.getElementById('app-engine');
            container.innerHTML = MASTER_PLAN.map(section => {
                const title = section.dynamic ? `🏠 ${ZONES[dayNum].name}` : section.title;
                const tasks = section.dynamic ? ZONES[dayNum].tasks : section.tasks;
                return `
                    <div class="glass-card p-5 accent-border">
                        <h3 class="text-[10px] font-extrabold text-emerald-400 uppercase tracking-widest mb-4 italic">${title}</h3>
                        <div class="space-y-3">
                            ${tasks.map(task => {
                                const id = btoa(title + task).slice(0, 10);
                                return `
                                    <div class="flex items-center gap-3">
                                        <input type="checkbox" id="${id}" onchange="saveToStorage()" ${getCheckState(id)}
                                            class="w-5 h-5 rounded bg-slate-800 border-slate-700 text-emerald-500 focus:ring-0">
                                        <label for="${id}" class="text-xs font-medium">${task}</label>
                                    </div>`;
                            }).join('')}
                        </div>
                    </div>`;
            }).join('');
        }

        function getCheckState(id) {
            const data = JSON.parse(localStorage.getItem(currentKey)) || { checks: {} };
            return data.checks[id] ? 'checked' : '';
        }

        function saveToStorage() {
            const checks = {};
            document.querySelectorAll('input[type="checkbox"]').forEach(cb => checks[cb.id] = cb.checked);
            const note = document.getElementById('daily-note').value;
            localStorage.setItem(currentKey, JSON.stringify({ checks, mood: window.currentMood, note }));
            updateStats();
        }

        function updateMood(m) {
            window.currentMood = m;
            updateMoodUI(m);
            saveToStorage();
        }

        function updateMoodUI(m) {
            document.querySelectorAll('.mood-btn').forEach(btn => btn.classList.remove('active'));
            if (m) {
                const idMap = { 'sad': 'm-sad', 'neutral': 'm-neutral', 'happy': 'm-happy' };
                document.getElementById(idMap[m]).classList.add('active');
            }
        }

        function updateStats() {
            const cbs = document.querySelectorAll('input[type="checkbox"]');
            const done = [...cbs].filter(c => c.checked).length;
            const percent = cbs.length ? Math.round((done / cbs.length) * 100) : 0;
            document.getElementById('clarity-percent').innerText = percent + "%";
            document.getElementById('longevity-score').innerText = (done * 0.05).toFixed(1);
        }

        function exportToExcel() {
            if (!window.currentMood) return alert("Selecciona un estado emocional antes de exportar.");
            
            const date = document.getElementById('date-picker').value;
            const longevity = document.getElementById('longevity-score').innerText;
            const clarity = document.getElementById('clarity-percent').innerText;
            const mood = window.currentMood;
            const note = document.getElementById('daily-note').value.replace(/,/g, " "); // Quitar comas para no romper el CSV

            // Crear contenido CSV
            const headers = "Fecha,Longevidad,Claridad,Mood,Nota\n";
            const row = `${date},${longevity},${clarity},${mood},${note}\n`;
            const blob = new Blob([headers + row], { type: 'text/csv;charset=utf-8;' });
            
            // Crear link de descarga
            const link = document.createElement("a");
            const url = URL.createObjectURL(blob);
            link.setAttribute("href", url);
            link.setAttribute("download", `EGIO_Corte_${date}.csv`);
            link.style.visibility = 'hidden';
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);

            alert("✅ Archivo CSV descargado. Puedes abrirlo en Excel.");
        }

        init();
    </script>
</body>
</html>
