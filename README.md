<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS PRO 2026 - Adaptive Fixed</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    
    <style>
        :root { --app-height: 100dvh; }
        
        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden;
            background-color: #0f172a;
            font-family: system-ui, -apple-system, sans-serif;
            display: grid;
            /* แบ่งสัดส่วน: บน(คงที่), กลาง(ยืดหยุ่น), ล่าง(คงที่) */
            grid-template-rows: auto 1fr auto; 
        }

        /* ปรับแต่ง Scrollbar ของ Log */
        .log-container {
            min-height: 0; /* สำคัญมาก: เพื่อไม่ให้ Grid แตก */
            overflow-y: auto;
            background-color: #f1f5f9;
            -webkit-overflow-scrolling: touch;
        }

        /* ปุ่มกดที่ปรับขนาดตามหน้าจออัตโนมัติ */
        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            height: 100%; /* ให้สูงเต็ม Grid Cell */
            min-height: 38px;
            font-size: clamp(9px, 2vw, 13px);
            font-weight: 800;
            border-radius: 8px;
            transition: all 0.1s;
            line-height: 1;
        }
        .drug-btn:active { transform: scale(0.95); opacity: 0.8; }

        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; color: #f43f5e; }
        }
        .blink-warning { animation: blink 0.8s infinite; color: #f43f5e !important; }

        #summary-screen {
            display: none; position: fixed; inset: 0;
            background: white; z-index: 100; overflow-y: auto;
            padding: env(safe-area-inset-top) 15px env(safe-area-inset-bottom) 15px;
        }
    </style>
</head>
<body>

    <!-- 1. ส่วนบน: Timers & Rhythm Selection (Fixed) -->
    <header class="bg-slate-900 text-white p-3 pt-[env(safe-area-inset-top)]">
        <div class="flex justify-between items-center mb-2">
            <h1 class="text-lg font-black text-rose-500 italic">ACLS 2026</h1>
            <div class="text-right">
                <p class="text-[8px] text-slate-500 font-bold uppercase">Total</p>
                <p id="total-timer" class="text-xl font-mono font-bold text-emerald-400">00:00</p>
            </div>
        </div>
        
        <div class="grid grid-cols-2 gap-2 mb-2">
            <div class="bg-slate-800 p-1.5 rounded-lg border-2 border-blue-500 text-center">
                <p id="cpr-timer" class="text-3xl font-mono font-black">02:00</p>
            </div>
            <div id="guidance-text" class="bg-slate-800 p-1.5 rounded-lg border border-slate-700 text-[10px] text-slate-400 italic flex items-center justify-center text-center">
                Standby...
            </div>
        </div>

        <button onclick="toggleStart()" id="btn-start" class="w-full bg-emerald-600 py-2.5 rounded-lg font-black text-xs uppercase shadow-md mb-2">Start Code Blue</button>

        <div class="grid grid-cols-4 gap-1.5">
            <button onclick="setRhythm('VF')" class="bg-rose-700 py-2 rounded-lg font-black text-[10px] uppercase">VF</button>
            <button onclick="setRhythm('pVT')" class="bg-rose-500 py-2 rounded-lg font-black text-[10px] uppercase">pVT</button>
            <button onclick="setRhythm('PEA')" class="bg-blue-700 py-2 rounded-lg font-black text-[10px] uppercase">PEA</button>
            <button onclick="setRhythm('Asystole')" class="bg-sky-600 py-2 rounded-lg font-black text-[10px] uppercase">Asystole</button>
        </div>
    </header>

    <!-- 2. ส่วนกลาง: Log (Flexible) -->
    <main class="log-container">
        <table class="w-full text-left border-separate border-spacing-y-1 px-2 py-1">
            <tbody id="log-body" class="text-[10px] font-bold text-slate-700"></tbody>
        </table>
    </main>

    <!-- 3. ส่วนล่าง: Drug Buttons & Footer (Fixed) -->
    <footer class="bg-white border-t border-slate-200 p-2 pb-[env(safe-area-inset-bottom)]">
        <div class="grid grid-cols-3 gap-1.5 mb-2 h-24"> <!-- บังคับความสูงส่วนปุ่มยาไว้เล็กน้อย -->
            <button onclick="recordAction('CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">CPR</button>
            <button onclick="recordAction('Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
            <button onclick="recordAction('IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
            
            <button onclick="recordDefib()" class="drug-btn bg-rose-600 text-white italic">SHOCK</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white">EPI</button>
            <button onclick="recordAction('Amio 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600">AMIO 300</button>
        </div>

        <div class="flex items-center gap-2">
            <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-3 rounded-xl font-black text-xs uppercase">ROSC</button>
            <div class="flex gap-1 flex-1">
                <div class="bg-rose-600 text-white flex-1 py-1 rounded-lg text-center leading-none">
                    <p class="text-[7px]">S</p><p id="dash-shock" class="text-base font-black">0</p>
                </div>
                <div class="bg-blue-600 text-white flex-1 py-1 rounded-lg text-center leading-none">
                    <p class="text-[7px]">E</p><p id="dash-epi" class="text-base font-black">0</p>
                </div>
            </div>
            <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white py-3 rounded-xl font-black text-[9px] uppercase">DEAD</button>
        </div>
    </footer>

    <!-- Summary Screen -->
    <div id="summary-screen">
        <div class="flex justify-between items-center border-b-2 border-slate-900 pb-2 mb-4">
            <h2 class="text-xl font-black italic uppercase">Summary</h2>
            <button onclick="location.reload()" class="text-rose-500 font-bold text-xs">RESET</button>
        </div>
        <div id="summary-stats" class="grid grid-cols-2 gap-2 mb-4"></div>
        <div id="sum-timeline" class="text-[11px] space-y-1"></div>
        <button onclick="document.getElementById('summary-screen').style.display='none'" class="w-full mt-6 py-3 bg-slate-900 text-white rounded-xl font-black uppercase text-xs">Close</button>
    </div>

    <script>
        let totalSec = 0, cprSec = 120, isRunning = false, mainInterval = null;
        let defibCount = 0, epiCount = 0, timelineData = [];

        function formatTime(s) {
            const m = Math.floor(s / 60).toString().padStart(2, '0');
            const sec = (s % 60).toString().padStart(2, '0');
            return `${m}:${sec}`;
        }

        function toggleStart() {
            const btn = document.getElementById('btn-start');
            if (!isRunning) {
                isRunning = true; btn.innerText = 'PAUSE'; btn.className = 'w-full bg-amber-600 py-2.5 rounded-lg font-black text-xs uppercase';
                mainInterval = setInterval(() => {
                    totalSec++; cprSec--;
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    const cprDisp = document.getElementById('cpr-timer');
                    cprDisp.innerText = formatTime(cprSec);
                    if (cprSec <= 15 && cprSec > 0) cprDisp.classList.add('blink-warning');
                    else cprDisp.classList.remove('blink-warning');
                    if (cprSec <= 0) { cprSec = 120; triggerAlert(); }
                }, 1000);
            } else {
                isRunning = false; btn.innerText = 'RESUME'; btn.className = 'w-full bg-emerald-600 py-2.5 rounded-lg font-black text-xs uppercase';
                clearInterval(mainInterval);
            }
        }

        function triggerAlert() {
            if ('speechSynthesis' in window) window.speechSynthesis.speak(new SpeechSynthesisUtterance("Rhythm Check."));
            recordAction('⏰ Rhythm Check Due!');
            Swal.fire({ title: 'RHYTHM CHECK!', icon: 'warning', confirmButtonText: 'RESUMED' });
        }

        function recordAction(msg) {
            const now = new Date();
            const realTime = now.toLocaleTimeString('th-TH', { hour12: false });
            const elapsed = formatTime(totalSec);
            timelineData.push({ realTime, elapsed, action: msg });
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm";
            row.innerHTML = `<td class="p-2 w-16 text-[8px] font-mono border-r">${realTime}<br>T+${elapsed}</td><td class="px-2 uppercase">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
        }

        function setRhythm(type) {
            const isShock = (type === 'VF' || type === 'pVT');
            document.getElementById('guidance-text').innerHTML = isShock ? `<b class="text-rose-500">SHOCK</b>` : `<b class="text-blue-500">EPI</b>`;
            recordAction(`Rhythm: ${type}`);
            cprSec = 120;
        }

        function recordDefib() {
            defibCount++; document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ SHOCK #${defibCount}`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++; document.getElementById('dash-epi').innerText = epiCount;
            recordAction(`💊 EPI #${epiCount}`);
        }

        function showSummary(outcome) {
            if (isRunning) toggleStart();
            document.getElementById('summary-screen').style.display = 'block';
            document.getElementById('summary-stats').innerHTML = `
                <div class="bg-slate-100 p-2 rounded-lg text-center"><p class="text-[8px] uppercase">Time</p><p class="text-lg font-black">${formatTime(totalSec)}</p></div>
                <div class="bg-slate-100 p-2 rounded-lg text-center"><p class="text-[8px] uppercase">Outcome</p><p class="text-sm font-black">${outcome}</p></div>
            `;
            const timelineBox = document.getElementById('sum-timeline');
            timelineBox.innerHTML = '';
            timelineData.forEach(item => {
                const div = document.createElement('div');
                div.className = "border-b border-slate-100 py-1 flex justify-between";
                div.innerHTML = `<span>${item.realTime}</span><span class="font-bold">${item.action}</span>`;
                timelineBox.appendChild(div);
            });
        }
    </script>
</body>
</html>
