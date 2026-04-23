<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>نظام مافيا أحمد المحترف</title>
    <style>
        /* التنسيق العام - أسود وأحمر (ستايل هاكرز) */
        * { touch-action: manipulation; box-sizing: border-box; user-select: none; }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: #000; color: #0f0; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; }
        .container { width: 100%; max-width: 450px; text-align: center; }
        h1 { color: #da291c; text-shadow: 0 0 10px #da291c; margin-bottom: 20px; }
        
        /* المدخلات والأزرار */
        input { width: 100%; padding: 15px; background: #111; border: 1px solid #0f0; color: #0f0; border-radius: 10px; text-align: center; font-size: 1.1em; margin-bottom: 10px; }
        button { width: 100%; padding: 15px; background: #da291c; color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; font-size: 1.1em; transition: 0.2s; }
        button:active { transform: scale(0.95); }
        .btn-start { background: #008000; margin-top: 10px; }

        /* عرض الأسماء المضافة */
        #preview { margin: 15px 0; color: #888; font-size: 0.9em; min-height: 20px; border-bottom: 1px solid #222; padding-bottom: 10px; }

        /* منطقة اللعب والكروت */
        .narrator-box { background: #0055ff; color: white; padding: 20px; border-radius: 12px; margin-bottom: 20px; font-weight: bold; border: 2px solid white; font-size: 1.2em; }
        .mafia-card { background: #1a1a1a; padding: 30px; margin: 12px 0; border-radius: 12px; border: 1px solid #444; cursor: pointer; position: relative; transition: 0.4s; }
        .role-text { display: none; color: #ff0000; font-size: 1.6em; font-weight: bold; text-shadow: 0 0 8px #000; }
        
        /* تأثيرات القفل والاختفاء */
        .locked-screen { pointer-events: none; opacity: 0.3; }
        .fade-out { opacity: 0; transform: scale(0.85); pointer-events: none; transition: 0.6s; }
        .hidden { display: none; }
    </style>
</head>
<body>

    <div class="container">
        <h1>AHMED MAFIA PRO</h1>

        <div id="setup-area">
            <input type="text" id="playerName" placeholder="أدخل اسم اللاعب..." autocomplete="off">
            <button onclick="addPlayer()">إضافة لاعب +</button>
            <div id="preview">اللاعبين: لا يوجد</div>
            <button class="btn-start" onclick="distributeRoles()">توزيع الأدوار والراوي</button>
        </div>

        <div id="play-area" class="hidden">
            <div id="narrator-display"></div>
            <p style="color: #666; font-size: 0.8em; margin-bottom: 15px;">اضغط على اسمك لرؤية دورك (يختفي بعد 3 ثواني)</p>
            <div id="cards-container"></div>
            <button style="background: #333; margin-top: 30px;" onclick="location.reload()">إعادة اللعبة</button>
        </div>
    </div>

    <script>
        let players = [];
        let isLocked = false;

        // إضافة لاعب للقائمة
        function addPlayer() {
            const input = document.getElementById('playerName');
            const name = input.value.trim();
            if (name) {
                players.push(name);
                document.getElementById('preview').innerText = "اللاعبين: " + players.join(" - ");
                input.value = "";
                input.focus();
            }
        }

        // توزيع الأدوار بناءً على قوانين أحمد
        function distributeRoles() {
            if (players.length < 4) return alert("لازم على الأقل 4 لاعبين (3 لاعبين + 1 راوي)!");

            // 1. اختيار الراوي وحذفه من التوزيع
            const narratorIndex = Math.floor(Math.random() * players.length);
            const narratorName = players[narratorIndex];
            const activePlayers = players.filter((_, i) => i !== narratorIndex);

            // 2. تحديد عدد المافيا (قانون أحمد المحترف)
            let roles = [];
            const pCount = activePlayers.length;

            if (pCount >= 10) {
                roles.push("مافيا القتل 🔪", "مافيا التسكيت 🤫"); // 2 مافيا
            } else {
                roles.push("مافيا القتل 💀"); // 1 مافيا
            }

            roles.push("دكتور 💉");
            while (roles.length < pCount) roles.push("مواطن 👨‍🌾");

            // خلط الأدوار عشوائياً
            roles.sort(() => Math.random() - 0.5);

            // 3. عرض النتائج
            document.getElementById('setup-area').classList.add('hidden');
            const playArea = document.getElementById('play-area');
            playArea.classList.remove('hidden');

            document.getElementById('narrator-display').innerHTML = `
                <div class="narrator-box">🎤 الراوي: ${narratorName}</div>
            `;

            const container = document.getElementById('cards-container');
            container.innerHTML = "";

            activePlayers.forEach((name, i) => {
                const card = document.createElement('div');
                card.className = 'mafia-card';
                card.id = `card-${i}`;
                card.innerHTML = `
                    <b id="name-${i}">${name}</b>
                    <div class="role-text" id="role-${i}">${roles[i]}</div>
                `;

                card.onclick = function() {
                    if (isLocked) return;

                    // قفل الغش: منع أي ضغطة ثانية
                    isLocked = true;
                    document.querySelectorAll('.mafia-card').forEach(c => c.classList.add('locked-screen'));
                    card.classList.remove('locked-screen');

                    document.getElementById(`name-${i}`).classList.add('hidden');
                    document.getElementById(`role-${i}`).style.display = 'block';

                    // إخفاء الكرت نهائياً بعد 3 ثواني
                    setTimeout(() => {
                        card.classList.add('fade-out');
                        setTimeout(() => {
                            card.style.visibility = 'hidden'; // حذف المساحة لكن الحفاظ على الترتيب
                            isLocked = false;
                            document.querySelectorAll('.mafia-card').forEach(c => c.classList.remove('locked-screen'));
                        }, 600);
                    }, 3000);
                };
                container.appendChild(card);
            });
        }
    </script>
</body>
</html>
