let mood = null;
let currentDateKey = "";

function getDateKey(dateObj) {
    return "egio_data_" + dateObj.toISOString().split('T')[0];
}

function loadDayData() {
    const picker = document.getElementById("date-picker");
    const selectedDate = new Date(picker.value);
    currentDateKey = getDateKey(selectedDate);

    const saved = JSON.parse(localStorage.getItem(currentDateKey)) || {
        checks: {},
        mood: null,
        note: ""
    };

    mood = saved.mood;
    document.getElementById("daily-note").value = saved.note || "";

    document.querySelectorAll('input[type="checkbox"]').forEach(c => {
        c.checked = saved.checks[c.id] || false;
    });

    highlightMood();
    updateStats();
}

function saveDayData() {
    const checks = {};
    document.querySelectorAll('input[type="checkbox"]').forEach(c => {
        checks[c.id] = c.checked;
    });

    const data = {
        checks,
        mood,
        note: document.getElementById("daily-note").value
    };

    localStorage.setItem(currentDateKey, JSON.stringify(data));
}

function highlightMood() {
    document.querySelectorAll('button[id^="m-"]').forEach(b => b.classList.add('grayscale'));
    if (!mood) return;

    const map = { happy: "hap", sad: "sad", neutral: "neu" };
    document.getElementById("m-" + map[mood]).classList.remove('grayscale');
}

function setMood(m) {
    mood = m;
    highlightMood();
    saveDayData();
}

function saveCheck(id) {
    saveDayData();
    updateStats();
}

function updateStats() {
    const checks = document.querySelectorAll('input[type="checkbox"]');
    const done = Array.from(checks).filter(c => c.checked).length;
    const percent = checks.length ? Math.round((done / checks.length) * 100) : 0;

    document.getElementById('clarity-percent').innerText = percent + '%';
    document.getElementById('longevity-score').innerText = (done * 0.05).toFixed(1);
}

function initDatePicker() {
    const picker = document.getElementById("date-picker");
    const today = new Date().toISOString().split('T')[0];
    picker.value = today;

    picker.addEventListener("change", loadDayData);
}

function initApp() {
    const now = new Date();
    const day = now.getDay();
    const hour = now.getHours();

    document.getElementById('header-date').innerText =
        now.toLocaleDateString('es-MX', { weekday: 'long', day: 'numeric', month: 'short' });

    OS_PLAN[3].tasks = DAILY_ZONES[day].tasks;
    OS_PLAN[3].title = `🏠 Micro-Zona: ${DAILY_ZONES[day].name}`;

    const container = document.getElementById('app-engine');

    container.innerHTML = OS_PLAN.map(block => {
        const startHour = parseInt(block.time.split(':')[0]);
        const isCurrent = hour >= startHour && hour < (startHour + 4);

        return `
        <div class="glass-card p-5 ${isCurrent ? 'current-block ring-1 ring-sky-500/30' : 'opacity-50'}">
            <div class="flex justify-between items-start mb-3">
                <div>
                    <h3 class="text-[10px] font-bold text-sky-400 uppercase tracking-tighter">${block.time}</h3>
                    <p class="font-bold text-md">${block.title}</p>
                </div>
            </div>
            <div class="space-y-2">
                ${block.tasks.map(task => {
                    const id = btoa(task).substring(0,10);
                    return `
                    <div class="flex items-center">
                        <input type="checkbox" id="${id}" onchange="saveCheck('${id}')"
                        class="w-4 h-4 rounded bg-slate-800 border-slate-700 text-sky-500 focus:ring-0">
                        <label for="${id}" class="ml-3 text-xs font-medium text-slate-300">${task}</label>
                    </div>`;
                }).join('')}
            </div>
        </div>`;
    }).join('');

    initDatePicker();
    loadDayData();
}

document.getElementById("daily-note").addEventListener("input", saveDayData);

initApp();
