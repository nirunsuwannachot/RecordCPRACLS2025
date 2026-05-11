# RecordCPRACLS2025
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS PRO 2026 - Universal Edition</title>
    
    <!-- Tailwind & SweetAlert2 -->
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
            /* ป้องกันการลากเพื่อ Refresh ในมือถือ */
            overscroll-behavior-y: contain;
        }

        /* Animation การกระพริบเตือน */
        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; color: #f43f5e; transform: scale(1.02); }
        }
        .blink-warning {
            animation: blink 0.8s infinite;
            color: #f43f5e !important;
        }

        /* ปรับความสูงของ Log ให้ยืดหยุ่นตามหน้าจอ */
        .log-container {
            flex-grow: 1;
            overflow-y: auto;
            background-color: #f8fafc;
            border-top: 1px solid #e2e8f0;
            -webkit-overflow-scrolling: touch;
        }

        /* ปุ่มขนาดใหญ่ขึ้นเพื่อการสัมผัส (Touch Friendly) */
        .action-btn {
            display: flex; align-items: center; justify-content: center;
            height: 54px; font-size: 11px; font-weight: 800;
            border-radius: 12px; transition: all 0.1s;
            box-shadow: 0 2px 0 rgba(0,0,0,0.1);
            user-select: none;
        }
        .action-btn:active { transform: scale(0.96); filter: brightness(0.9); }

        /* iPad Optimization */
        @media (min-width: 768px) {
            .action-btn { height: 70px; font-size: 16px; }
            #cpr-timer { font-size: 5rem !important; }
            #total-timer { font-size: 3rem !important; }
        }

        /* ซ่อน Scrollbar */
        .no-scrollbar::-webkit-scrollbar { display: none; }
    </style>
</head>
<body>

    <div id="main-app" class="flex flex-col h-full safe-area-inset">
        <!-- TOP PANEL: Timers & Stats -->
        <div class="p-4 bg-slate-900 text-white shadow-2xl z-20">
            <div class="flex justify-between items-end mb-3">
                <div>
                    <h1 class="text-xl md:text-3xl font-black text-rose-500 italic leading-none">ACLS PRO</h1>
                    <span id="status-tag" class="text-[9px] font-bold bg-slate-800 text-slate-500 px-2 py-0.5 rounded-full">STANDBY</span>
                </div>
                <div class="text-right">
                    <p class="text-[9px] text-slate-500 font-bold uppercase">Total Duration</p>
                    <p id="total-timer" class="text-3xl md:text-5xl font-mono font-black text-emerald-400 leading-none">00:00</p>
                </div>
            </div>
            
            <div class="grid grid-cols-2 gap-3 mb-4">
                <div class="bg-slate-800 p-3 rounded-2xl border-2 border-blue-500 text-center shadow-inner">
                    <p class="text-[10px] text-blue-400 font-bold uppercase mb-1">Next Rhythm Check</p>
                    <p id="cpr-timer" class="text-4xl md:text-7xl font-mono font-black">02:00</p>
                </div>
                <div class="bg-slate-800 p-3 rounded-2xl border border-slate-700 flex flex-col justify-center">
                    <p class="text-amber-500 text-[10px] font-bold uppercase mb-1">Guidance</p>
                    <div id="guidance-text" class="text-slate-400 text-xs italic leading-snug">Press Start to Begin...</div>
                </div>
            </div>

            <!-- Start / Rhythm Controls -->
            <button onclick="toggleStart()" id="btn-start" class="w-full bg-emerald-600 h-14 md:h-20 mb-4 rounded-2xl font-black text-lg uppercase shadow-lg border-b-4 border-emerald-800 active:border-b-0 transition-all">Start Code Blue</button>

            <div class="grid grid-cols-2 gap-3">
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="setRhythm('VF')" class="bg-rose-700 h-12 md:h-16 rounded-xl font-black text-xs md:text-lg border-b-4 border-rose-950 active:border-b-0 uppercase">VF</button>
                    <button onclick="setRhythm('pVT')" class="bg-rose-500 h-12 md:h-16 rounded-xl font-black text-xs md:text-lg border-b-4 border-rose-800 active:border-b-0 uppercase">pVT</button>
                </div>
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="setRhythm('PEA')" class="bg-blue-700 h-12 md:h-16 rounded-xl font-black text-xs md:text-lg border-b-4 border-blue-950 active:border-b-0 uppercase">PEA</button>
                    <button onclick="setRhythm('Asystole')" class="bg-sky-600 h-12 md:h-16 rounded-xl font-black text-xs md:text-lg border-b-4 border-sky-800 active:border-b-0 uppercase">Asystole</button>
                </div>
            </div>
        </div>

        <!-- ACTION PANEL: Meds & Procedures -->
        <div class="p-3 bg-white grid grid-cols-3 gap-2 shadow-inner">
            <button onclick="recordAction('CPR Started')" class="action-btn border-2 border-emerald-500 text-emerald-700 font-black">CPR START</button>
            <button onclick="recordAction('Access IV/IO')" class="action-btn border-2 border-sky-600 text-sky-700 font-black">IV/IO</button>
            <button onclick="recordAction('IV Fluid Bolus')" class="action-btn border-2 border-blue-400 text-blue-700 font-black">FLUID</button>
            
            <button onclick="recordDefib()" class="action-btn bg-rose-600 text-white col-span-1 italic font-black text-sm">SHOCK 200J</button>
            <button onclick="recordEpi()" class="action-btn bg-blue-600 text-white font-black text-sm">EPINEPHRINE</button>
            <button onclick="recordAction('Amio 300mg')" class="action-btn border-2 border-purple-500 text-purple-700 font-black">AMIO 300</button>
            
            <button onclick="recordAction('Amio 150mg')" class="action-btn bg-slate-200 text-slate-700 font-black">AMIO 150</button>
            <button onclick="recordAction('Advanced Airway')" class="action-btn bg-orange-100 text-orange-700 border border-orange-200 col-span-2 font-black italic">ADV. AIRWAY</button>
        </div>

        <!-- LOG PANEL -->
        <div class="log-container no-scrollbar">
            <div class="sticky top-0 bg-slate-100/90 backdrop-blur px-4 py-1 text-[10px] font-black text-slate-400 uppercase flex justify-between z-10 border-b">
                <span>Timestamp</span>
                <span>Interventions</span>
            </div>
            <table class="w-full text-left border-separate border-spacing-y-1.5 px-3 py-2">
                <tbody id="log-body" class="text-[11px] md:text-sm font-bold text-slate-700"></tbody>
            </table>
        </div>

        <!-- FOOTER: Outcomes -->
        <div class="p-4 bg-white border-t-2 border-slate-100 flex items-center gap-3">
            <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white h-16 rounded-2xl font-black text-lg shadow-lg">ROSC</button>
            <div class="flex gap-2 flex-1">
                <div class="bg-rose-600 text-white flex-1 py-1 rounded-xl text-center shadow-md">
                    <p class="text-[8px] font-black uppercase mt-1">Defib</p>
                    <p id="dash-shock" class="text-2xl font-black">0</p>
                </div>
                <div class="bg-blue-600 text-white flex-1 py-1 rounded-xl text-center shadow-md">
                    <p class="text-[8px] font-black uppercase mt-1">Epi</p>
                    <p id="dash-epi" class="text-2xl font-black">0</p>
                </div>
            </div>
            <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white h-16 rounded-2xl font-black text-xs uppercase shadow-md">DEAD</button>
        </div>
    </div>

    <!-- SUMMARY OVERLAY -->
    <div id="summary-screen">
        <div class="max-w-2xl mx-auto">
            <div class="flex justify-between items-center border-b-4 border-slate-900 pb-4 mb-6">
                <h2 class="text-3xl font-black italic text-slate-900 uppercase">Case Summary</h2>
                <button onclick="location.reload()" class="bg-rose-500 text-white px-6 py-2 rounded-full font-black text-xs uppercase shadow-lg">Reset</button>
            </div>
            <div id="summary-stats" class="grid grid-cols-2 gap-4 mb-8"></div>
            <h3 class="font-black text-slate-900 uppercase text-xs mb-3 flex items-center gap-2">
                <div class="w-1.5 h-4 bg-amber-500"></div> Case Timeline (Real-time)
            </h3>
            <div id="sum-timeline" class="space-y-2 mb-10"></div>
            <button onclick="document.getElementById('summary-screen').style.display='none'" class="w-full py-5 bg-slate-900 text-white rounded-2xl font-black uppercase tracking-widest mb-10 shadow-xl">Back to Records</button>
        </div>
    </div>

    <script>
        let totalSec = 0, cprSec = 120, isRunning = false, mainInterval = null;
        let defibCount = 0, epiCount = 0, timelineData = [];

        function formatTime(s) {
            const m = Math.floor(Math.abs(s) / 60).toString().padStart(2, '0');
            const sec = (Math.abs(s) % 60).toString().padStart(2, '0');
            return `${m}:${sec}`;
        }

        function toggleStart() {
            const btn = document.getElementById('btn-start');
            const status = document.getElementById('status-tag');
            
            if (!isRunning) {
                isRunning = true;
                btn.innerText = 'PAUSE CODE';
                btn.className = 'w-full bg-amber-600 h-14 md:h-20 mb-4 rounded-2xl font-black text-lg uppercase shadow-lg border-b-4 border-amber-800 transition-all';
                status.innerText = 'ACTIVE';
                status.className = 'text-[9px] font-bold bg-rose-500 text-white px-2 py-0.5 rounded-full animate-pulse';
                
                if (totalSec === 0) recordAction('Code Blue Activated');
                
                mainInterval = setInterval(() => {
                    totalSec++; 
                    cprSec--;
                    
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    const cprDisp = document.getElementById('cpr-timer');
                    cprDisp.innerText = formatTime(cprSec);

                    // 15s Blink Warning
                    if (cprSec <= 15 && cprSec > 0) cprDisp.classList.add('blink-warning');
                    else cprDisp.classList.remove('blink-warning');

                    if (cprSec <= 0) {
                        cprSec = 120;
                        triggerAlert();
                    }
                }, 1000);
            } else {
                isRunning = false;
                btn.innerText = 'RESUME CODE';
                btn.className = 'w-full bg-emerald-600 h-14 md:h-20 mb-4 rounded-2xl font-black text-lg uppercase shadow-lg border-b-4 border-emerald-800 transition-all';
                status.innerText = 'PAUSED';
                status.className = 'text-[9px] font-bold bg-slate-700 text-slate-300 px-2 py-0.5 rounded-full';
                clearInterval(mainInterval);
            }
        }

        function triggerAlert() {
            if ('speechSynthesis' in window) {
                const msg = new SpeechSynthesisUtterance("Time is up. Check Rhythm.");
                msg.rate = 1.1;
                window.speechSynthesis.speak(msg);
            }
            if (navigator.vibrate) navigator.vibrate([400, 200, 400]);
            
            recordAction('⏰ RHYTHM CHECK DUE');
            
            Swal.fire({
                title: 'RHYTHM CHECK!',
                html: '<p class="text-xl font-bold text-rose-600">Check Pulse & EKG</p>Consider Switching Compressor',
                icon: 'warning',
                confirmButtonText: 'CPR RESUMED',
                confirmButtonColor: '#0f172a',
                timer: 15000,
                timerProgressBar: true
            });
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
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm overflow-hidden";
            row.innerHTML = `
                <td class="text-center font-mono py-2 bg-slate-50 border-r w-24">
                    <div class="text-slate-900 font-bold">${realTime}</div>
                    <div class="text-[8px] text-slate-400 mt-0.5 uppercase tracking-tighter">T+${elapsed}</div>
                </td>
                <td class="px-4 py-2 font-black text-slate-800">${msg}</td>
            `;
            body.insertBefore(row, body.firstChild);
        }

        function setRhythm(type) {
            const isShock = (type === 'VF' || type === 'pVT');
            document.getElementById('guidance-text').innerHTML = isShock ? 
                `<b class="text-rose-600 text-sm uppercase">Shock 200J</b><br><span class="text-slate-400">Restart CPR Immediately</span>` : 
                `<b class="text-blue-600 text-sm uppercase">Check Epi</b><br><span class="text-slate-400">Restart CPR Immediately</span>`;
            recordAction(`Rhythm: ${type} (${isShock ? 'Shockable' : 'Non-shock'})`);
            cprSec = 120;
            if (!isRunning) toggleStart();
        }

        function recordDefib() {
            defibCount++; 
            document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ DEFIB #${defibCount} (200J)`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++; 
            document.getElementById('dash-epi').innerText = epiCount;
            recordAction(`💊 EPINEPHRINE #${epiCount}`);
        }

        function showSummary(outcome) {
            if (isRunning) toggleStart();
            document.getElementById('summary-screen').style.display = 'block';

            document.getElementById('summary-stats').innerHTML = `
                <div class="bg-slate-900 text-white p-4 rounded-3xl shadow-lg">
                    <p class="text-[10px] text-slate-400 font-bold uppercase">Duration</p>
                    <p class="text-3xl font-mono font-black text-emerald-400">${formatTime(totalSec)}</p>
                </div>
                <div class="bg-slate-100 p-4 rounded-3xl border-2 border-slate-200">
                    <p class="text-slate-500 text-[10px] font-bold uppercase">Outcome</p>
                    <p class="text-xl font-black italic ${outcome==='ROSC'?'text-emerald-600':'text-slate-800'}">${outcome}</p>
                </div>
                <div class="bg-rose-50 p-4 rounded-3xl border-2 border-rose-200 text-center">
                    <p class="text-rose-600 text-[10px] font-bold uppercase">Total Shock</p>
                    <p class="text-3xl font-black text-rose-700">${defibCount}</p>
                </div>
                <div class="bg-blue-50 p-4 rounded-3xl border-2 border-blue-200 text-center">
                    <p class="text-blue-600 text-[10px] font-bold uppercase">Total Epi</p>
                    <p class="text-3xl font-black text-blue-700">${epiCount}</p>
                </div>
            `;

            const timelineBox = document.getElementById('sum-timeline');
            timelineBox.innerHTML = '';
            timelineData.forEach(item => {
                const div = document.createElement('div');
                div.className = "flex gap-4 items-center bg-slate-50 p-3 rounded-2xl border-l-4 border-slate-300";
                div.innerHTML = `<span class="font-mono text-slate-400 text-xs w-20">[${item.realTime}]</span><span class="text-sm font-black text-slate-700">${item.action}</span>`;
                timelineBox.appendChild(div);
            });
        }
    </script>
</body>
</html>
