<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EGIO PROGRESS | Life Architecture</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f3f4f6; color: #1f2937; }
        .card { background: white; border-radius: 16px; padding: 1.5rem; shadow: 0 10px 15px -3px rgba(0,0,0,0.1); }
        .btn-status { transition: all 0.2s; filter: grayscale(1); opacity: 0.5; }
        .btn-status.active { filter: grayscale(0); opacity: 1; transform: scale(1.2); }
    </style>
</head>
<body class="max-w-4xl mx-auto p-4 pb-20">

    <header class="flex justify-between items-center my-6">
        <div>
            <h1 class="text-3xl font-extrabold tracking-tight text-slate-900">EGIO <span class="text-blue-600">PROGRESS</span></h1>
            <p id="current-date" class="text-sm text-slate-500 font-medium uppercase tracking-widest"></p>
        </div>
        <div class="text-right">
            <div class="text-xs text-slate-400">LONGEVIDAD ACUMULADA</div>
            <div class="text-2xl font-bold text-emerald-600">+<span id="years-gained">0</span> días</div>
        </div>
    </header>

    <main class="grid gap-6">
        
        <section class="card border-l-8 border-blue-500">
            <h2 class="text-lg font-bold mb-3 flex items-center">🏠 Micro-Zona: <span id="daily-zone" class="ml-2 text-blue-600"></span></h2>
            <div id="tasks-container" class="space-y-3 text-sm">
                </div>
        </section>

        <section class="card border-l-8 border-emerald-500">
            <h2 class="text-lg font-bold mb-4 italic">📊 PeakWatch & Reseña</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                    <label class="block text-xs font-bold text-slate-400 mb-1 uppercase">Estado de Ánimo</label>
                    <div class="flex gap-4 text-3xl">
                        <button onclick="setStatus('sad')" id="btn-sad" class="btn-status">😢</button>
                        <button onclick="setStatus('neutral')" id="btn-neutral" class="btn-status">😐</button>
                        <button onclick="setStatus('happy')" id="btn-happy" class="btn-status">😊</button>
                    </div>
                </div>
                <div>
                    <label class="block text-xs font-bold text-slate-400 mb-1 uppercase">Nota del Día (Breve)</label>
                    <input type="text" id="day-note" placeholder="¿Cómo fue tu enfoque hoy?" class="w-full border-b-2 border-slate-200 py-1 focus:border-emerald-500 outline-none transition-colors">
                </div>
            </div>
            <button onclick="saveEntry()" class="w-full mt-6 bg-slate-900 text-white py-3 rounded-xl font-bold hover:bg-slate-800 transition-all">FINALIZAR DÍA & GUARDAR</button>
        </section>

        <section class="mt-8">
            <h3 class="text-sm font-bold text-slate-500 uppercase tracking-widest mb-4">Historial de Evolución</h3>
            <div id="history-container" class="space-y-3">
                </div>
        </section>

    </main>

    <script>
        const zones = {
            1: { name: "Habitación (Orden Visual)", tasks: ["Organizar ropa", "Limpiar superficies", "Revisar outfit semanal"] },
            2: { name: "Cocina (Fuego Limpio)", tasks: ["Limpiar estufa", "Revisar refri", "Ordenar despensa"] },
            3: { name: "Baño (Sanitización)", tasks: ["Limpiar lavabo", "Espejo", "Desinfección rápida"] },
            4: { name: "Área Común (Flujo)", tasks: ["Barrer/Trapear", "Ordenar mesa/escritorio"] },
            5: { name: "Reset General", tasks: ["Vaciar basuras", "Gestionar ropa sucia", "Revisión visual"] },
            6: { name: "Cocina + Meal Prep", tasks: ["Preparar alimentos semana", "Limpieza profunda"] },
            0: { name: "Mente + Planificación", tasks: ["Revisar semana", "Checklist EGIO", "Cierre mental"] }
        };

        let currentStatus = null;
        let entries = JSON.parse(localStorage.getItem('egio_entries')) || [];

        function init() {
            const now = new Date();
            document.getElementById('current-date').innerText = now.toLocaleDateString('es-ES', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
            
            const dayNum = now.getDay();
            const zone = zones[dayNum];
            document.getElementById('daily-zone').innerText = zone.name;
            
            const container = document.getElementById('tasks-container');
            container.innerHTML = zone.tasks.map(t => `
                <label class="flex items-center p-2 hover:bg-slate-50 rounded-lg cursor-pointer">
                    <input type="checkbox" class="w-4 h-4 text-blue-600 rounded border-gray-300">
                    <span class="ml-3">${t}</span>
                </label>
            `).join('');

            updateHistory();
            calculateLongevity();
        }

        function setStatus(s) {
            currentStatus = s;
            document.querySelectorAll('.btn-status').forEach(b => b.classList.remove('active'));
            document.getElementById('btn-' + s).classList.add('active');
        }

        function saveEntry() {
            if(!currentStatus) return alert("Por favor, selecciona tu estado de ánimo.");
            
            const newEntry = {
                id: Date.now(),
                date: new Date().toLocaleString(),
                status: currentStatus,
                note: document.getElementById('day-note').value,
                zone: document.getElementById('daily-zone').innerText
            };

            entries.unshift(newEntry);
            localStorage.setItem('egio_entries', JSON.stringify(entries));
            
            // Reset UI
            document.getElementById('day-note').value = '';
            currentStatus = null;
            document.querySelectorAll('.btn-status').forEach(b => b.classList.remove('active'));
            
            updateHistory();
            calculateLongevity();
        }

        function deleteEntry(id) {
            entries = entries.filter(e => e.id !== id);
            localStorage.setItem('egio_entries', JSON.stringify(entries));
            updateHistory();
            calculateLongevity();
        }

        function updateHistory() {
            const container = document.getElementById('history-container');
            container.innerHTML = entries.map(e => `
                <div class="card flex justify-between items-center shadow-sm py-3 px-4">
                    <div class="flex items-center gap-4">
                        <span class="text-2xl">${e.status === 'happy' ? '😊' : e.status === 'neutral' ? '😐' : '😢'}</span>
                        <div>
                            <p class="text-[10px] text-slate-400 uppercase font-bold">${e.date}</p>
                            <p class="text-sm font-semibold text-slate-700">${e.note || 'Sin nota'}</p>
                        </div>
                    </div>
                    <button onclick="deleteEntry(${e.id})" class="text-red-300 hover:text-red-500 text-xs">Borrar</button>
                </div>
            `).join('');
        }

        function calculateLongevity() {
            // Lógica: Cada entrada guardada suma 0.5 días de "longevidad" por disciplina
            const days = (entries.length * 0.5).toFixed(1);
            document.getElementById('years-gained').innerText = days;
        }

        init();
    </script>
</body>
</html>
