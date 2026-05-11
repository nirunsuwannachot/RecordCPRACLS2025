<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS PRO 2026 - Universal Adaptive</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    
    <style>
        :root { --app-height: 100dvh; }
        
        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            background-color: #0f172a;
            font-family: system-ui, -apple-system, sans-serif;
            overscroll-behavior-y: contain;
        }

        /* ส่วนที่คงที่: Timer, ปุ่มกด, Footer */
        .fixed-panel { flex-shrink: 0; }

        /* ส่วนที่ยืดหยุ่น: Log จะขยายตามพื้นที่ที่เหลือ */
        .log-container {
            flex-grow: 1;
            overflow-y: auto;
            background-color: #f1f5f9;
            border-top: 2px solid #e2e8f0;
            -webkit-overflow-scrolling: touch;
        }

        /* ปุ่มกดแบบ Adaptive */
        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            height: clamp(42px, 6vh, 65px); 
            font-size: clamp(10px, 2.5vw, 14px); 
            font-weight: 800;
            border-radius: 10px; transition: all 0.1s;
            user-select: none;
        }
        .drug-btn:active { transform: scale(0.96); opacity: 0.8; }

        /* Animation การกระพริบเตือน */
        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; color: #f43f5e; transform: scale(1.02); }
        }
        .blink-warning { animation: blink 0.8s infinite; color: #f43f5e !important; }

        #summary-screen {
            display: none; position: fixed; inset: 0;
            background: white; z-index: 100; overflow-y: auto;
            padding: env(safe-area-inset-top) 20px env(safe-area-inset-bottom) 20px;
        }

        /* สำหรับหน้าจอใหญ่เช่น iPad */
        @media (min-width: 768px) {
            .drug-btn { font-size: 16px; }
            #cpr-timer { font-size: 5rem !important; }
        }
    </style>
</head>
<body>

    <!-- 1. Header Panel (Fixed) -->
    <div class="fixed-panel p-3 bg-slate-900 text-white pt-[env(safe-area-inset-top)]">
        <div class="flex justify-between items-center mb-2">
            <h1 class="text-xl font-black text-rose-500 italic">ACLS 2026</h1>
            <div class="text-right">
                <p class="text-[8px] text-slate-500 font-bold uppercase">Total Duration</p>
                <p id="total-timer" class="text-2xl font-mono font-bold text-emerald-400">00:00</p>
            </div>
        </div>
        
        <div class="grid grid-cols-2 gap-2 mb-3">
            <div class="bg-slate-800 p-2 rounded-xl border-2 border-blue-500 text-center shadow-inner">
                <p class="text-[8px] text-blue-400 font-bold uppercase">Next Rhythm Check</p>
                <p id="cpr-timer" class="text-4xl font-mono font-black">02:00</p>
            </div>
            <div class="bg-slate-800 p-2 rounded-xl border border-slate-700 flex flex-col justify-center px-2">
                <p class="text-amber-500 text-[9px] font-bold uppercase">Guidance</p>
                <div id="guidance-text" class="text-slate-400 text-[10px] italic leading-tight">Ready...</div>
            </div>
        </div>

        <button onclick="toggleStart()" id="btn-start" class="w-full bg-emerald-600 py-3 mb-3 rounded-xl font-black text-sm uppercase shadow-lg border-b-4 border-emerald-800 active:border-b-0">Start Code Blue</button>

        <div class="grid grid-cols-2 gap-2">
            <div class="grid grid-cols-2 gap-2">
                <button onclick="setRhythm('VF')" class="bg-rose-700 py-2 rounded-xl font-black text-xs uppercase border-b-4 border-rose-950">VF</button>
                <button onclick="setRhythm('pVT')" class="bg-rose-500 py-2 rounded-xl font-black text-xs uppercase border-b-4 border-rose-800">pVT</button>
            </div>
            <div class="grid grid-cols-2 gap-2">
                <button onclick="setRhythm('PEA')" class="bg-blue-700 py-2 rounded-xl font-black text-xs uppercase border-b-4 border-blue-950">PEA</button>
                <button onclick="setRhythm('Asystole')" class="bg-sky-600 py-2 rounded-xl font-black text-xs uppercase border-b-4 border-sky-800">Asystole</button>
            </div>
        </div>
    </div>

    <!-- 2. Action Panel (Fixed) -->
    <div class="fixed-panel p-2.5 bg-white border-b border-slate-200 shadow-sm">
        <div class="grid grid-cols-3 gap-1.5 mb-1.5">
            <button onclick="recordAction('CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">START CPR</button>
            <button onclick="recordAction('Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
            <button onclick="recordAction('IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
        </div>
        <div class="grid grid-cols-3 gap-1.5 mb-1.5">
            <button onclick="recordDefib()" class="drug-btn bg-rose-600 text-white shadow-md italic">DEFIB 200J</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPINEPHRINE</button>
            <button onclick="recordAction('Amiodarone 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600 uppercase">Amio 300</button>
        </div>
        <div class="grid grid-cols-2 gap-1.5">
            <button onclick="recordAction('Amiodarone 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700 uppercase">Amio 150</button>
            <button onclick="recordAction('Advanced Airway')" class="drug-btn bg-orange-50 border border-orange-200 text-orange-700 italic uppercase">Adv. Airway</button>
        </div>
    </div>

    <!-- 3. Log Container (Flexible) -->
    <div class="log-container">
        <table class="w-full text-left border-separate border-spacing-y-1 px-3 py-2">
            <thead class="sticky top-0 bg-slate-100 z-10">
                <tr class="text-[9px] text-slate-400 font-black uppercase">
                    <th class="py-1 w-20 text-center border-r">Real-Time</th>
                    <th class="py-1 px-2">Action</th>
                </tr>
            </thead>
            <tbody id="log-body" class="text-[10px] md:text-sm font-bold text-slate-700"></tbody>
        </table>
    </div>

    <!-- 4. Footer Panel (Fixed) -->
    <div class="fixed-panel bg-white border-t p-3 flex items-center gap-3 pb-[calc(env(safe-area-inset-bottom)+12px)]">
        <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-4 rounded-xl font-black text-xs uppercase shadow-lg">ROSC</button>
        <div class="flex gap-1.5 flex-1">
            <div class="bg-rose-600 text-white flex-1 py-1 rounded-xl text-center"><p class="text-[7px] uppercase mt-1">Defib</p><p id="dash-shock" class="text-xl font-black">0</p></div>
            <div class="bg-blue-600 text-white flex-1 py-1 rounded-xl text-center"><p class="text-[7px] uppercase mt-1">Epi</p><p id="dash-epi" class="text-xl font-black">0</p></div>
        </div>
        <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white py-4 rounded-xl font-black text-[9px] uppercase">DEAD</button>
    </div>

    <!-- Summary Screen -->
    <div id="summary-screen">
        <div class="flex justify-between items-center border-b-2 border-slate-900 pb-3 mb-4">
            <h2 class="text-2xl font-black italic text-slate-900 uppercase tracking-tight">Case Summary</h2>
            <button onclick="location.reload()" class="bg-rose-500 text-white px-4 py-1 rounded-full font-bold text-xs">NEW CASE</button>
        </div>
        <div id="summary-stats" class="grid grid-cols-2 gap-4 mb-6"></div>
        <h3 class="font-black text-slate-900 uppercase text-xs mb-2">Timeline History</h3>
        <div id="sum-timeline" class="space-y-2 mb-8 border-t pt-2"></div>
        <button onclick="document.getElementById('summary-screen').style.display='none'" class="w-full py-4 border-2 border-slate-900 rounded-xl font-black uppercase text-sm">Close</button>
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
                isRunning = true; btn.innerText = 'PAUSE'; btn.className = 'w-full bg-amber-600 py-3 mb-3 rounded-xl font-black text-sm uppercase shadow-lg transition-all';
                if (totalSec === 0) recordAction('Code Blue Activated');
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
                isRunning = false; btn.innerText = 'RESUME'; btn.className = 'w-full bg-emerald-600 py-3 mb-3 rounded-xl font-black text-sm uppercase shadow-lg transition-all';
                clearInterval(mainInterval);
            }
        }

        function triggerAlert() {
            if ('speechSynthesis' in window) window.speechSynthesis.speak(new SpeechSynthesisUtterance("Time is up. Rhythm Check."));
            recordAction('⏰ Rhythm Check Due!');
            Swal.fire({ title: 'RHYTHM CHECK!', text: 'Check Pulse & EKG', icon: 'warning', confirmButtonText: 'RESUMED', confirmButtonColor: '#0f172a' });
        }

        function recordAction(msg) {
            const now = new Date();
            const realTime = now.getHours().toString().padStart(2, '0') + ':' + 
                             now.getMinutes().toString().padStart(2, '0') + ':' + 
                             now.getSeconds().toString().padStart(2, '0');
            const elapsed = formatTime(totalSec);
            timelineData.push({ realTime, elapsed, action: msg });
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm";
            row.innerHTML = `<td class="text-center font-mono py-2 bg-slate-50 border-r leading-none">${realTime}<br><span class="text-[7px] text-slate-400">T+${elapsed}</span></td><td class="px-3 uppercase font-black text-slate-800">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
        }

        function setRhythm(type) {
            const isShock = (type === 'VF' || type === 'pVT');
            document.getElementById('guidance-text').innerHTML = isShock ? `<b class="text-rose-500 uppercase">Shock 200J</b> -> CPR` : `<b class="text-blue-500 uppercase">Give Epi</b> -> CPR`;
            recordAction(`Rhythm: ${type}`);
            cprSec = 120;
        }

        function recordDefib() {
            defibCount++; document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ DEFIB #${defibCount} (200J)`);
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
                <div class="bg-slate-100 p-3 rounded-xl"><p class="text-[9px] text-slate-500 font-bold uppercase">Time</p><p class="text-xl font-black">${formatTime(totalSec)}</p></div>
                <div class="bg-slate-100 p-3 rounded-xl"><p class="text-[9px] text-slate-500 font-bold uppercase">Outcome</p><p class="text-lg font-black">${outcome}</p></div>
                <div class="bg-rose-50 p-3 rounded-xl border border-rose-200 text-center"><p class="text-rose-600 text-[9px] font-bold uppercase">Shock</p><p class="text-2xl font-black text-rose-700">${defibCount}</p></div>
                <div class="bg-blue-50 p-3 rounded-xl border border-blue-200 text-center"><p class="text-blue-600 text-[9px] font-bold uppercase">Epi</p><p class="text-2xl font-black text-blue-700">${epiCount}</p></div>
            `;
            const timelineBox = document.getElementById('sum-timeline');
            timelineBox.innerHTML = '';
            timelineData.forEach(item => {
                const div = document.createElement('div');
                div.className = "flex gap-3 border-b border-slate-100 pb-1";
                div.innerHTML = `<span class="font-mono text-slate-400 text-[9px] w-24">[${item.realTime}]</span><span class="text-[10px] font-black text-slate-700 uppercase">${item.action}</span>`;
                timelineBox.appendChild(div);
            });
        }
    </script>
</body>
</html>
