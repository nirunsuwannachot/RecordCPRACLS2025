<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS PRO 2026 - Scrollable Log</title>
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

        /* ส่วนบนคงที่ */
        .fixed-header { flex-shrink: 0; }

        /* ส่วน Log: จำกัดความสูงไว้ที่ 5 รายการ (ประมาณ 160px - 180px) */
        .log-section {
            height: 165px; /* ความสูงคงที่สำหรับประมาณ 5 บรรทัด */
            flex-shrink: 0;
            overflow-y: auto;
            background-color: #f1f5f9;
            border-top: 2px solid #e2e8f0;
            border-bottom: 2px solid #e2e8f0;
            -webkit-overflow-scrolling: touch;
        }

        /* ส่วนปุ่มกด: ให้ยืดหยุ่นตามพื้นที่ที่เหลือด้านล่าง */
        .action-footer {
            flex-grow: 1;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 8px;
            background-color: white;
            min-height: 0;
        }

        .drug-btn {
            display: flex; align-items: center; justify-content: center;
            height: 100%;
            min-height: 40px;
            font-size: clamp(10px, 2.5vw, 13px);
            font-weight: 800;
            border-radius: 8px;
            transition: all 0.1s;
        }
        .drug-btn:active { transform: scale(0.95); opacity: 0.8; }

        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; color: #f43f5e; }
        }
        .blink-warning { animation: blink 0.8s infinite; color: #f43f5e !important; }

        /* ซ่อน Scrollbar สำหรับความสวยงาม (ไม่บังคับ) */
        .no-scrollbar::-webkit-scrollbar { display: none; }
    </style>
</head>
<body>

    <!-- ส่วนบน: Timers & Rhythm -->
    <header class="fixed-header p-3 bg-slate-900 text-white pt-[env(safe-area-inset-top)]">
        <div class="flex justify-between items-center mb-2">
            <h1 class="text-lg font-black text-rose-500 italic">ACLS 2026</h1>
            <p id="total-timer" class="text-xl font-mono font-bold text-emerald-400">00:00</p>
        </div>
        
        <div class="grid grid-cols-2 gap-2 mb-2">
            <div class="bg-slate-800 p-2 rounded-lg border-2 border-blue-500 text-center">
                <p id="cpr-timer" class="text-3xl font-mono font-black">02:00</p>
            </div>
            <div id="guidance-text" class="bg-slate-800 p-2 rounded-lg border border-slate-700 text-[10px] text-slate-400 italic flex items-center justify-center text-center leading-tight">
                Standby...
            </div>
        </div>

        <button onclick="toggleStart()" id="btn-start" class="w-full bg-emerald-600 py-2 rounded-lg font-black text-xs uppercase mb-2 shadow-lg">Start Code Blue</button>

        <div class="grid grid-cols-4 gap-1.5">
            <button onclick="setRhythm('VF')" class="bg-rose-700 py-2 rounded-lg font-black text-[10px]">VF</button>
            <button onclick="setRhythm('pVT')" class="bg-rose-500 py-2 rounded-lg font-black text-[10px]">pVT</button>
            <button onclick="setRhythm('PEA')" class="bg-blue-700 py-2 rounded-lg font-black text-[10px]">PEA</button>
            <button onclick="setRhythm('Asystole')" class="bg-sky-600 py-2 rounded-lg font-black text-[10px]">Asystole</button>
        </div>
    </header>

    <!-- ส่วนกลาง: Log Table (Scroll ได้เฉพาะในนี้) -->
    <div class="log-section no-scrollbar" id="log-scroll">
        <table class="w-full text-left border-separate border-spacing-y-1 px-3 py-1">
            <tbody id="log-body" class="text-[11px] font-bold text-slate-700">
                <!-- ข้อมูลจะแสดงที่นี่ -->
            </tbody>
        </table>
    </div>

    <!-- ส่วนล่าง: Drug Buttons & Finish -->
    <footer class="action-footer pb-[env(safe-area-inset-bottom)]">
        <div class="grid grid-cols-3 gap-1.5 h-full mb-2">
            <button onclick="recordAction('CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">CPR</button>
            <button onclick="recordAction('Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
            <button onclick="recordAction('IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
            
            <button onclick="recordDefib()" class="drug-btn bg-rose-600 text-white italic">SHOCK</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPI</button>
            <button onclick="recordAction('Amio 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600">AMIO 300</button>
            
            <button onclick="recordAction('Amio 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700">AMIO 150</button>
            <button onclick="recordAction('Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-200 text-orange-700 italic col-span-2">ADV. AIRWAY</button>
        </div>

        <div class="flex items-center gap-2 mt-auto pt-1 border-t">
            <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-3.5 rounded-xl font-black text-xs uppercase shadow-md">ROSC</button>
            <div class="flex gap-1 flex-1">
                <div class="bg-rose-600 text-white flex-1 py-1 rounded-lg text-center"><p class="text-[7px]">S</p><p id="dash-shock" class="text-lg font-black leading-none">0</p></div>
                <div class="bg-blue-600 text-white flex-1 py-1 rounded-lg text-center"><p class="text-[7px]">E</p><p id="dash-epi" class="text-lg font-black leading-none">0</p></div>
            </div>
            <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white py-3.5 rounded-xl font-black text-[9px] uppercase">DEAD</button>
        </div>
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
                isRunning = true; btn.innerText = 'PAUSE'; btn.className = 'w-full bg-amber-600 py-2 rounded-lg font-black text-xs uppercase mb-2';
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
                isRunning = false; btn.innerText = 'RESUME'; btn.className = 'w-full bg-emerald-600 py-2 rounded-lg font-black text-xs uppercase mb-2';
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
            const realTime = now.getHours().toString().padStart(2, '0') + ':' + now.getMinutes().toString().padStart(2, '0');
            const elapsed = formatTime(totalSec);
            timelineData.push({ realTime, elapsed, action: msg });
            
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm";
            row.innerHTML = `<td class="p-2 w-16 text-[9px] font-mono border-r">${realTime}<br>T+${elapsed}</td><td class="px-3 uppercase font-black">${msg}</td>`;
            
            // เพิ่มรายการใหม่ไว้ด้านบนสุด
            body.insertBefore(row, body.firstChild);
            
            // สั่งให้ scroll กลับไปด้านบนสุดเพื่อให้เห็นรายการล่าสุดเสมอ
            document.getElementById('log-scroll').scrollTop = 0;
        }

        function setRhythm(type) {
            const isShock = (type === 'VF' || type === 'pVT');
            document.getElementById('guidance-text').innerHTML = isShock ? `<b class="text-rose-500 uppercase">SHOCKABLE</b><br>Shock 200J` : `<b class="text-blue-500 uppercase">NON-SHOCK</b><br>Give Epi`;
            recordAction(`Rhythm: ${type}`);
            cprSec = 120;
        }

        function recordDefib() {
            defibCount++; document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ DEFIB #${defibCount}`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++; document.getElementById('dash-epi').innerText = epiCount;
            recordAction(`💊 EPI #${epiCount}`);
        }

        function showSummary(outcome) {
            // โค้ดสรุปเหมือนเดิม
            Swal.fire({
                title: 'Case Summary',
                html: `Outcome: ${outcome}<br>Time: ${formatTime(totalSec)}<br>Shocks: ${defibCount}<br>Epi: ${epiCount}`,
                confirmButtonText: 'New Case',
                preConfirm: () => location.reload()
            });
        }
    </script>
</body>
</html>
