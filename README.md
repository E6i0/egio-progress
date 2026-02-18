<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EGIO PROGRESS | Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f8fafc; }
        .card { background: white; border-radius: 12px; padding: 1.5rem; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1); }
    </style>
</head>
<body class="p-6">
    <header class="mb-8 flex justify-between items-center">
        <div>
            <h1 class="text-3xl font-bold text-slate-800">EGIO PROGRESS</h1>
            <p class="text-slate-500 italic">Orden Físico = Claridad Mental</p>
        </div>
        <div class="text-right">
            <span id="biometric-badge" class="bg-green-100 text-green-700 px-3 py-1 rounded-full text-sm font-semibold italic">HRV: Óptimo</span>
        </div>
    </header>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <section class="card border-t-4 border-blue-500">
            <h2 class="text-xl font-semibold mb-4 flex items-center">🏠 Eje 1: Entorno</h2>
            <div class="space-y-4">
                <div>
                    <p class="text-xs text-slate-400 uppercase tracking-wider">Zona del Día</p>
                    <p class="font-medium text-lg text-blue-600">Z1: Centro de Control (Escritorio)</p>
                </div>
                <ul class="space-y-2 text-sm text-slate-600">
                    <li><input type="checkbox" class="mr-2"> Limpiar superficies</li>
                    <li><input type="checkbox" class="mr-2"> Gestión de cables</li>
                    <li><input type="checkbox" class="mr-2"> Reset de basura digital</li>
                </ul>
            </div>
        </section>

        <section class="card border-t-4 border-emerald-500">
            <h2 class="text-xl font-semibold mb-4 flex items-center">💰 Eje 2: Batching</h2>
            <div class="space-y-4">
                <div>
                    <p class="text-xs text-slate-400 uppercase tracking-wider">Fase Actual</p>
                    <p class="font-medium text-lg text-emerald-600">Producción Intensiva</p>
                </div>
                <div class="w-full bg-slate-100 rounded-full h-2">
                    <div class="bg-emerald-500 h-2 rounded-full" style="width: 45%"></div>
                </div>
                <p class="text-xs text-slate-500 italic">Meta: 3 videos editados hoy.</p>
            </div>
        </section>

        <section class="card border-t-4 border-purple-500">
            <h2 class="text-xl font-semibold mb-4 flex items-center">🤝 Eje 3: Foco Social</h2>
            <div class="space-y-4 text-sm">
                <div class="flex justify-between items-center bg-purple-50 p-3 rounded-lg">
                    <span>Deep Work Timer</span>
                    <span class="font-mono font-bold text-purple-700">50:00</span>
                </div>
                <p class="text-slate-600">Próxima ventana de networking: 16:00 hrs (Asíncrono).</p>
            </div>
        </section>
    </div>

    <footer class="mt-12 p-6 bg-slate-800 text-white rounded-xl flex justify-around items-center">
        <div class="text-center">
            <p class="text-2xl font-bold">85%</p>
            <p class="text-xs uppercase text-slate-400 italic">Claridad Mental</p>
        </div>
        <div class="h-10 w-px bg-slate-600"></div>
        <div class="text-center">
            <p class="text-2xl font-bold">$120</p>
            <p class="text-xs uppercase text-slate-400 italic">Profit / Hora Foco</p>
        </div>
    </footer>
</body>
</html>
