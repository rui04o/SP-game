<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لعبة محاكاة الألواح الشمسية</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: #fff;
            text-align: center;
            margin: 0;
            padding: 20px;
            overflow: hidden;
        }
        h1 { margin-bottom: 5px; color: #f1c40f; text-shadow: 2px 2px 4px rgba(0,0,0,0.5); }
        p { margin-top: 5px; font-size: 1.1rem; }
        #game-container {
            position: relative;
            width: 800px;
            height: 500px;
            margin: 20px auto;
            background: linear-gradient(to bottom, #87CEEB, #E0F6FF, #2c3e50);
            border: 4px solid #fff;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            overflow: hidden;
            cursor: none; /* إخفاء مؤشر الفأرة داخل اللعبة */
        }
        /* اللوح الشمسي */
        #solar-panel {
            position: absolute;
            bottom: 40px;
            left: 360px;
            width: 90px;
            height: 20px;
            background: linear-gradient(to bottom, #005580, #002233);
            border: 2px solid #00d2ff;
            border-radius: 4px;
            box-shadow: 0 0 10px #00d2ff;
            /* خطوط شبكة السيليكون */
            background-image: linear-gradient(90deg, transparent 45%, rgba(255,255,255,0.3) 50%, transparent 55%);
            background-size: 15px 100%;
        }
        /* قاعدة اللوح الشمسي */
        #panel-stand {
            position: absolute;
            bottom: 0;
            left: 400px;
            width: 10px;
            height: 40px;
            background: #7f8c8d;
        }
        /* أشعة الشمس */
        .sun-ray {
            position: absolute;
            width: 16px;
            height: 16px;
            background: radial-gradient(circle, #fff 10%, #ffdb13 60%, #f39c12 100%);
            border-radius: 50%;
            box-shadow: 0 0 12px #ffdb13;
        }
        /* واجهة العدادات */
        #ui {
            display: flex;
            justify-content: space-between;
            width: 800px;
            margin: 0 auto;
            background: rgba(0, 0, 0, 0.4);
            padding: 15px;
            border-radius: 10px;
            font-size: 1.2rem;
            box-sizing: border-box;
        }
        .battery-box {
            display: flex;
            align-items: center;
            gap: 10px;
        }
        #battery-bar {
            width: 150px;
            height: 25px;
            border: 2px solid #fff;
            border-radius: 5px;
            padding: 2px;
            background: rgba(0,0,0,0.5);
        }
        #battery-fill {
            width: 0%;
            height: 100%;
            background: linear-gradient(90deg, #2ecc71, #27ae60);
            border-radius: 3px;
            transition: width 0.1s;
        }
        /* شاشات البداية والنهاية */
        .screen {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.85);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
        }
        button {
            padding: 12px 30px;
            font-size: 1.2rem;
            background-color: #27ae60;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
            transition: 0.2s;
        }
        button:hover { background-color: #2ecc71; transform: scale(1.05); }
        .hidden { display: none !important; }
    </style>
</head>
<body>

    <h1>☀️ مشروع تقنيات الطاقة: محاكي الألواح الشمسية</h1>
    <p>حرك اللوح الشمسي يميناً ويساراً بالفأرة لالتقاط أشعة الشمس وشحن بطارية المدينة!</p>

    <div id="ui">
        <div>⏳ الوقت المتبقي: <span id="time-txt">30</span> ثانية</div>
        <div class="battery-box">
            <span>🔋 شحن البطارية:</span>
            <div id="battery-bar"><div id="battery-fill"></div></div>
            <div><span id="score-txt">0</span>%</div>
        </div>
    </div>

    <div id="game-container">
        <!-- شاشة البداية -->
        <div id="start-screen" class="screen">
            <h2>محاكاة توليد الطاقة الشمسية</h2>
            <p>هدف المشروع: توضيح كيفية امتصاص الخلايا الكهروضوئية للضوء لتخزين الطاقة.</p>
            <button onclick="startGame()">ابدأ المحاكاة</button>
        </div>

        <!-- شاشة النهاية -->
        <div id="end-screen" class="screen hidden">
            <h2 id="end-title">انتهت المحاكاة!</h2>
            <p id="end-msg"></p>
            <button onclick="startGame()">إعادة التجربة</button>
        </div>

        <!-- عناصر اللعبة الإنشائية -->
        <div id="panel-stand"></div>
        <div id="solar-panel"></div>
    </div>

    <p style="font-size: 0.9rem; color: #bdc3c7;">عمل الطالب - مادة التقنية (الصف الأول الثانوي)</p>

    <script>
        const container = document.getElementById('game-container');
        const panel = document.getElementById('solar-panel');
        const stand = document.getElementById('panel-stand');
        const batteryFill = document.getElementById('battery-fill');
        const scoreTxt = document.getElementById('score-txt');
        const timeTxt = document.getElementById('time-txt');
        const startScreen = document.getElementById('start-screen');
        const endScreen = document.getElementById('end-screen');
        const endTitle = document.getElementById('end-title');
        const endMsg = document.getElementById('end-msg');

        let score = 0;
        let timeLeft = 30;
        let gameActive = false;
        let gameInterval, spawnInterval;
        let rays = [];

        // حركة اللوح مع الفأرة
        container.addEventListener('mousemove', (e) => {
            if (!gameActive) return;
            const rect = container.getBoundingClientRect();
            let x = e.clientX - rect.left - (panel.offsetWidth / 2);
            
            // منع اللوح من الخروج عن الحدود
            if (x < 0) x = 0;
            if (x > container.offsetWidth - panel.offsetWidth) x = container.offsetWidth - panel.offsetWidth;
            
            panel.style.left = x + 'px';
            stand.style.left = (x + (panel.offsetWidth/2) - 5) + 'px';
        });

        function startGame() {
            // إعادة ضبط الإعدادات
            score = 0;
            timeLeft = 30;
            gameActive = true;
            scoreTxt.innerText = score;
            timeTxt.innerText = timeLeft;
            batteryFill.style.width = '0%';
            
            // تنظيف أي أشعة قديمة
            rays.forEach(ray => ray.remove());
            rays = [];

            startScreen.classList.add('hidden');
            endScreen.classList.add('hidden');

            // بدء المؤقتات
            gameInterval = setInterval(updateGame, 20);
            spawnInterval = setInterval(spawnRay, 600); // توليد شعاع كل 0.6 ثانية
        }

        function spawnRay() {
            if (!gameActive) return;
            const ray = document.createElement('div');
            ray.classList.add('sun-ray');
            ray.style.left = Math.random() * (container.offsetWidth - 20) + 'px';
            ray.style.top = '0px';
            container.appendChild(ray);
            rays.push(ray);
        }

        function updateGame() {
            // 1. تحديث مؤشر الوقت كل ثانية تقريباً (بناء على تقليل الوقت)
            // لتسهيل الحساب، سنعتمد على عداد تناقص موازي
            
            // 2. تحريك الأشعة لأسفل وفحص الاصطدام
            for (let i = rays.length - 1; i >= 0; i--) {
                let ray = rays[i];
                let top = parseFloat(ray.style.top);
                top += 4; // سرعة سقوط الأشعة
                ray.style.top = top + 'px';

                // فحص إذا وصلت الأشعة للأرض
                if (top > container.offsetHeight) {
                    ray.remove();
                    rays.splice(i, 1);
                    continue;
                }

                // فحص الاصطدام باللوح الشمسي
                if (checkCollision(ray, panel)) {
                    score += 5; // زيادة الشحن عند الالتقاط
                    if (score > 100) score = 100;
                    scoreTxt.innerText = score;
                    batteryFill.style.width = score + '%';
                    
                    ray.remove();
                    rays.splice(i, 1);
                }
            }
        }

        // عداد الوقت المنفصل
        let timerCountdown = setInterval(() => {
            if (gameActive) {
                timeLeft--;
                timeTxt.innerText = timeLeft;
                if (timeLeft <= 0) {
                    endGame();
                }
            }
        }, 1000);

        function checkCollision(ray, panel) {
            const rRect = ray.getBoundingClientRect();
            const pRect = panel.getBoundingClientRect();

            return !(
                rRect.top > pRect.bottom ||
                rRect.bottom < pRect.top ||
                rRect.right < pRect.left ||
                rRect.left > pRect.right
            );
        }

        function endGame() {
            gameActive = false;
            clearInterval(gameInterval);
            clearInterval(spawnInterval);
            
            endScreen.classList.remove('hidden');
            if (score >= 80) {
                endTitle.innerText = "⚡ تم شحن المدينة بنجاح!";
                endTitle.style.color = "#2ecc71";
