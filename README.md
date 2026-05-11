<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS PRO 2026 - Control Top</title>
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
            display: flex;
            flex-direction: column;
        }

        /* ส่วนบน: หัวข้อและ Timer */
        .header-section { flex-shrink: 0; }

        /* ส่วนกลาง: แผงปุ่มรักษา (ให้พื้นที่เยอะขึ้น) */
        .treatment-section {
            flex-shrink: 0;
            background-color: white;
            padding: 12px;
            border-bottom: 2px solid #e2e8f0;
        }

        /* ส่วนล่าง: บันทึกรายการ (Log) - จำกัดความสูงและ Scroll ได้ */
        .log-section {
            flex-grow: 1; /* กินพื้นที่ที่เหลือ */
            min-height: 0;
            overflow-y: auto;
            background-color: #f8fafc;
            -webkit-overflow-scrolling: touch;
        }

        .drug-btn {
            display: flex; align-items: center; justify-content: center;
            height: 48px; /* ความสูงมาตรฐานให้กดง่าย */
            font-size: clamp(10px, 3vw, 13px);
            font-weight: 800;
            border-radius: 10px;
            transition: all 0.1s;
            box-shadow: 0 2px 0 rgba(0,0,0,0.1);
        }
        .drug-btn:active { transform: translateY(2px); box-shadow: none; opacity: 0.8; }

        .rhythm-btn {
            height: 38px;
            font-size: 11px;
            font-weight: 900;
            border-radius: 6px;
        }

        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; color: #f43f5e; }
        }
        .blink-warning { animation: blink 0.8s infinite; color: #f43f5e !important; }

        /* ตกแต่ง Log */
        .log-row {
            background: white;
            border-left: 4px solid #cbd5e1;
            margin: 4px 10px;
            padding: 8px;
            border-radius: 4px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 1px 2px rgba(0,0,0,0.05);
        }
    </style>
</head>
<body>

    <!-- 1. HEADER (Timer & Rhythm Selection) -->
    <header class="header-section p-3 bg-slate-900 text-white pt-[env(safe-area-inset-top)]">
        <div class="flex justify-between items-center mb-2">
            <h1 class="text-lg font-black text-rose-500 italic">ACLS 2026</h1>
            <p id="total-timer" class="text-xl font-mono font-bold text-emerald-400">00:00</p>
        </div>
        
        <div class="grid grid-cols-2 gap-2 mb-3">
            <div class="bg-slate-800 p-2 rounded-lg border-2 border-blue-500 text-center">
                <p id="cpr-timer" class="text-3xl font-mono font-black">02:00</p>
            </div>
            <div id="guidance-text" class="bg-slate-800 p-2 rounded-lg border border-slate-700 text-[10px] text-slate-400 italic flex items-center justify-center text-center">
                Select Rhythm to Start
            </div>
        </div>

        <button onclick="toggleStart()" id="btn-start" class="w-full bg-emerald-600 py-2.5 rounded-lg font-black text-xs uppercase mb-3 shadow-lg">Start Code Blue</button>

        <div class="grid grid-cols-4 gap-1.5">
            <button onclick="setRhythm('VF')" class="rhythm-btn bg-rose-700">VF</button>
            <button onclick="setRhythm('pVT')" class="rhythm-btn bg-rose-500">pVT</button>
            <button onclick="setRhythm('PEA')" class="rhythm-btn bg-blue-700">PEA</button>
            <button onclick="setRhythm('Asystole')" class="rhythm-btn bg-sky-600">Asystole</button>
        </div>
    </header>

    <!-- 2. TREATMENT (Drug Buttons - อยู่บน Log แล้ว) -->
    <div class="treatment-section">
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordAction('CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">CPR</button>
            <button onclick="recordAction('Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
            <button onclick="recordAction('IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
        </div>
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordDefib()" class="drug-btn bg-rose-600 text-white italic">SHOCK</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPI 1mg</button>
            <button onclick="recordAction('Amio 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600">AMIO 300</button>
        </div>
        <div class="grid grid-cols-2 gap-2">
            <button onclick="recordAction('Amio 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700">AMIO 150</button>
            <button onclick="recordAction('Adv. Airway')" class="drug-btn bg-orange-100 border border-orange-300 text-orange-800 italic">ADV. AIRWAY</button>
        </div>
    </div>

    <!-- 3. LOG (Record History - อยู่ล่างสุด) -->
    <div class="log-section" id="log-container">
        <div class="p-2 text-[10px] uppercase font-bold text-slate-400 sticky top-0 bg-[#f8fafc] z-10">Timeline Record</div>
        <div id="log-body">
            <!-- New entries appear here -->
        </div>
    </div>

    <!-- 4. FOOTER (Summary Buttons) -->
    <footer class="bg-white border-t p-3 flex items-center gap-3 pb-[calc(env(safe-area-inset-bottom)+10px)]">
        <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-4 rounded-xl font-black text-xs uppercase shadow-lg">ROSC</button>
        <div class="flex gap-1.5 flex-1">
            <div class="bg-rose-600 text-white flex-1 py-1 rounded-xl text-center"><p class="text-[7px]">SHOCK</p><p id="dash-shock" class="text-xl font-black">0</p></div>
            <div class="bg-blue-600 text-white flex-1 py-1 rounded-xl text-center"><p class="text-[7px]">EPI</p><p id="dash-epi" class="text-xl font-black">0</p></div>
        </div>
        <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white py-4 rounded-xl font-black text-[9px] uppercase">DEAD</button>
    </footer>

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
                isRunning = true; btn.innerText = 'PAUSE'; btn.className = 'w-full bg-amber-600 py-2.5 rounded-lg font-black text-xs uppercase shadow-lg';
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
                isRunning = false; btn.innerText = 'RESUME'; btn.className = 'w-full bg-emerald-600 py-2.5 rounded-lg font-black text-xs uppercase shadow-lg';
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
            const realTime = now.toLocaleTimeString('th-TH', { hour12: false, hour: '2-digit', minute: '2-digit' });
            const elapsed = formatTime(totalSec);
            timelineData.push({ realTime, elapsed, action: msg });
            
            const logBody = document.getElementById('log-body');
            const div = document.createElement('div');
            div.className = "log-row";
            div.innerHTML = `
                <div class="flex flex-col">
                    <span class="text-[11px] font-black text-slate-800 uppercase">${msg}</span>
                    <span class="text-[8px] text-slate-400 font-bold">Total: ${elapsed}</span>
                </div>
                <div class="text-[10px] font-mono text-slate-500">${realTime}</div>
            `;
            // เพิ่มรายการใหม่ไว้บนสุดของ Log Section
            logBody.insertBefore(div, logBody.firstChild);
        }

        function setRhythm(type) {
            const isShock = (type === 'VF' || type === 'pVT');
            document.getElementById('guidance-text').innerHTML = isShock ? 
                `<b class="text-rose-500 uppercase">SHOCKABLE</b><br>Shock 200J & CPR` : 
                `<b class="text-blue-500 uppercase">NON-SHOCK</b><br>Epi 1mg & CPR`;
            recordAction(`Rhythm: ${type}`);
            if (!isRunning) toggleStart();
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
            Swal.fire({
                title: 'Case Summary',
                html: `<b>Outcome: ${outcome}</b><br>Time: ${formatTime(totalSec)}<br>Shocks: ${defibCount}<br>Epi: ${epiCount}`,
                confirmButtonText: 'New Case',
                preConfirm: () => location.reload()
            });
        }
    </script>
</body>
</html>
