<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS PRO 2026 - Fixed Record</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    
    <style>
        :root { --app-height: 100dvh; }
        
        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden; /* ห้ามเลื่อนทั้งหน้าจอเพื่อความนิ่ง */
            background-color: #0f172a;
            font-family: system-ui, -apple-system, sans-serif;
            display: flex;
            flex-direction: column;
        }

        /* ส่วนบนและส่วนปุ่มกด (ความสูงตามเนื้อหา) */
        .fixed-top-panel { flex-shrink: 0; z-index: 10; }

        /* ส่วน Log: ล็อคความสูงไว้สำหรับ 5 รายการ (ประมาณ 180px) */
        .log-section {
            height: 180px; /* บังคับความสูงคงที่สำหรับ 5 รายการล่าสุด */
            flex-shrink: 0;
            overflow-y: auto;
            background-color: #f1f5f9;
            border-top: 2px solid #e2e8f0;
            border-bottom: 2px solid #cbd5e1;
            -webkit-overflow-scrolling: touch;
        }

        /* ส่วนปุ่มล่างสุด (ชิดขอบล่างเสมอ) */
        .fixed-bottom-panel {
            flex-grow: 1; /* ให้ส่วนนี้กินพื้นที่ที่เหลือด้านล่าง */
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
            background-color: white;
            padding-bottom: env(safe-area-inset-bottom);
        }

        .drug-btn {
            display: flex; align-items: center; justify-content: center;
            height: 48px; font-weight: 800; border-radius: 10px;
            font-size: clamp(10px, 3vw, 13px);
            transition: all 0.1s;
        }
        .drug-btn:active { transform: scale(0.96); opacity: 0.8; }

        .log-row {
            background: white; border-left: 4px solid #3b82f6;
            margin: 4px 10px; padding: 6px 10px; border-radius: 4px;
            display: flex; justify-content: space-between; align-items: center;
            box-shadow: 0 1px 2px rgba(0,0,0,0.05);
        }

        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; color: #f43f5e; }
        }
        .blink-warning { animation: blink 0.8s infinite; color: #f43f5e !important; }
        
        /* ซ่อน Scrollbar แต่ยังเลื่อนได้ (เพื่อความคลีน) */
        .log-section::-webkit-scrollbar { width: 0; background: transparent; }
    </style>
</head>
<body>

    <!-- [ส่วนที่ 1] ส่วนบน: Timer + ปุ่มรักษา -->
    <div class="fixed-top-panel">
        <!-- Header -->
        <header class="p-3 bg-slate-900 text-white pt-[env(safe-area-inset-top)]">
            <div class="flex justify-between items-center mb-2">
                <h1 class="text-lg font-black text-rose-500 italic uppercase">ACLS PRO</h1>
                <p id="total-timer" class="text-xl font-mono font-bold text-emerald-400">00:00</p>
            </div>
            <div class="grid grid-cols-2 gap-2 mb-2">
                <div class="bg-slate-800 p-2 rounded-lg border-2 border-blue-500 text-center">
                    <p id="cpr-timer" class="text-3xl font-mono font-black">02:00</p>
                </div>
                <div id="guidance-text" class="bg-slate-800 p-2 rounded-lg border border-slate-700 text-[10px] text-slate-400 italic flex items-center justify-center text-center">
                    Standby...
                </div>
            </div>
            <div class="grid grid-cols-4 gap-1.5 mb-2">
                <button onclick="setRhythm('VF')" class="bg-rose-700 py-1.5 rounded-md font-bold text-[10px]">VF</button>
                <button onclick="setRhythm('pVT')" class="bg-rose-500 py-1.5 rounded-md font-bold text-[10px]">pVT</button>
                <button onclick="setRhythm('PEA')" class="bg-blue-700 py-1.5 rounded-md font-bold text-[10px]">PEA</button>
                <button onclick="setRhythm('Asystole')" class="bg-sky-600 py-1.5 rounded-md font-bold text-[10px]">Asystole</button>
            </div>
            <button onclick="toggleStart()" id="btn-start" class="w-full bg-emerald-600 py-2.5 rounded-lg font-black text-xs uppercase shadow-lg">Start Code Blue</button>
        </header>

        <!-- แผงปุ่มรักษาหลัก (คงที่ ไม่ขยับ) -->
        <div class="bg-white p-3 border-b-2 border-slate-200">
            <div class="grid grid-cols-3 gap-2 mb-2">
                <button onclick="recordAction('CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">CPR</button>
                <button onclick="recordAction('IV/IO Access')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
                <button onclick="recordAction('IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
            </div>
            <div class="grid grid-cols-3 gap-2 mb-2">
                <button onclick="recordDefib()" class="drug-btn bg-rose-600 text-white italic shadow-md">SHOCK</button>
                <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPI 1mg</button>
                <button onclick="recordAction('Amio 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600">AMIO 300</button>
            </div>
            <div class="grid grid-cols-2 gap-2">
                <button onclick="recordAction('Amio 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700">AMIO 150</button>
                <button onclick="recordAction('Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-200 text-orange-800 italic">ADV. AIRWAY</button>
            </div>
        </div>
    </div>

    <!-- [ส่วนที่ 2] LOG: บันทึก (ล็อคความสูง 5 รายการ) -->
    <div class="log-section" id="log-scroll-area">
        <div class="sticky top-0 bg-slate-100 px-4 py-1 text-[9px] font-black text-slate-400 uppercase tracking-widest border-b">
            History (Latest on top)
        </div>
        <div id="log-body" class="py-1">
            <!-- บันทึกจะแสดงที่นี่ -->
        </div>
    </div>

    <!-- [ส่วนที่ 3] FOOTER: ปุ่ม ROSC / DEAD (อยู่ล่างสุดเสมอ) -->
    <div class="fixed-bottom-panel p-3">
        <div class="flex items-center gap-3">
            <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-4 rounded-xl font-black text-xs uppercase shadow-lg">ROSC</button>
            <div class="flex gap-1.5 flex-1">
                <div class="bg-rose-600 text-white flex-1 py-1 rounded-lg text-center leading-none">
                    <p class="text-[7px] mb-1">SHOCK</p>
                    <p id="dash-shock" class="text-xl font-black">0</p>
                </div>
                <div class="bg-blue-600 text-white flex-1 py-1 rounded-lg text-center leading-none">
                    <p class="text-[7px] mb-1">EPI</p>
                    <p id="dash-epi" class="text-xl font-black">0</p>
                </div>
            </div>
            <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white py-4 rounded-xl font-black text-[9px] uppercase">DEAD</button>
        </div>
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
            Swal.fire({ title: 'RHYTHM CHECK!', icon: 'warning', confirmButtonText: 'OK' });
        }

        function recordAction(msg) {
            const now = new Date();
            const realTime = now.getHours().toString().padStart(2, '0') + ':' + now.getMinutes().toString().padStart(2, '0');
            const elapsed = formatTime(totalSec);
            timelineData.push({ realTime, elapsed, action: msg });
            
            const logBody = document.getElementById('log-body');
            const div = document.createElement('div');
            div.className = "log-row";
            div.innerHTML = `
                <div class="flex flex-col">
                    <span class="text-[10px] font-black text-slate-800 uppercase leading-tight">${msg}</span>
                    <span class="text-[7px] text-slate-400 font-bold">T+ ${elapsed}</span>
                </div>
                <div class="text-[9px] font-mono text-slate-500">${realTime}</div>
            `;
            logBody.insertBefore(div, logBody.firstChild);
            
            // เลื่อน Log กลับไปบนสุดอัตโนมัติเพื่อให้เห็นรายการล่าสุด
            document.getElementById('log-scroll-area').scrollTop = 0;
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

        function setRhythm(type) {
            const isShock = (type === 'VF' || type === 'pVT');
            document.getElementById('guidance-text').innerHTML = isShock ? 
                `<b class="text-rose-500 uppercase">Shockable</b><br>Defib 200J & CPR` : 
                `<b class="text-blue-500 uppercase">Non-Shockable</b><br>Give Epi & CPR`;
            recordAction(`Rhythm: ${type}`);
            if(!isRunning) toggleStart();
        }

        function showSummary(outcome) {
            if (isRunning) toggleStart();
            Swal.fire({
                title: 'Case Summary',
                html: `<b>Outcome: ${outcome}</b><br>Total Time: ${formatTime(totalSec)}<br>Shocks: ${defibCount}<br>Epi: ${epiCount}`,
                showCancelButton: true,
                confirmButtonText: 'Reset New Case',
                cancelButtonText: 'Review Log'
            }).then((result) => {
                if (result.isConfirmed) location.reload();
            });
        }
    </script>
</body>
</html>
