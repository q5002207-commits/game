<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>بوابة أحمد المحترف - النسخة المعدلة</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.css">
    <style>
        * { touch-action: manipulation; user-select: none; box-sizing: border-box; }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: #000; color: #0f0; margin: 0; padding: 10px; display: flex; flex-direction: column; align-items: center; overflow-x: hidden; }
        #main-menu { text-align: center; margin-top: 40px; width: 100%; max-width: 400px; }
        .menu-btn { display: block; width: 100%; padding: 25px; margin: 15px 0; background: #111; border: 2px solid #da291c; color: white; border-radius: 15px; font-size: 1.4em; font-weight: bold; cursor: pointer; transition: 0.3s; }
        .menu-btn:hover { background: #da291c; box-shadow: 0 0 20px #da291c; }
        .back-home { background: #333; color: #fff; padding: 8px 15px; border-radius: 5px; cursor: pointer; margin-bottom: 10px; border: none; font-size: 0.8em; }
        .header { width: 100%; max-width: 400px; background: #da291c; padding: 12px; border-radius: 12px; text-align: center; color: white; margin-bottom: 10px; }
        #board { width: 380px !important; max-width: 95vw !important; border: 4px solid #222; border-radius: 8px; }
        input { width: 100%; padding: 12px; border-radius: 8px; border: 1px solid #0f0; background: #000; color: #0f0; margin-bottom: 8px; text-align: center; }
        button { width: 100%; padding: 12px; background: #da291c; color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; }
        img[class^="piece"] { border-radius: 50% !important; border: 1px solid white !important; }
        .mafia-card, .spy-card { background: #1a1a1a; padding: 20px; margin: 10px 0; border-radius: 10px; border: 1px solid #444; transition: 0.5s; cursor: pointer; text-align: center; }
        .role-display { display: none; color: #ff0000; font-size: 1.2em; font-weight: bold; margin-top: 10px; }
        .narrator-box { background: #0055ff; color: white; padding: 15px; border-radius: 10px; margin-bottom: 15px; font-weight: bold; border: 2px solid white; }
        .hidden { display: none; }
        .fade-out { opacity: 0; transform: scale(0.9); pointer-events: none; }
        .locked { pointer-events: none; opacity: 0.4; }
    </style>
</head>
<body>

    <div id="main-menu">
        <h1 style="color:#da291c;">AHMED PRO SYSTEM</h1>
        <button class="menu-btn" onclick="showGame('chess')">♟️ ساحة الشطرنج</button>
        <button class="menu-btn" onclick="showGame('mafia')">🕵️ موزع المافيا (قانون أحمد)</button>
        <button class="menu-btn" onclick="showGame('spy')">🔍 لعبة الجاسوس </button>
    </div>

    <div id="chess-section" class="hidden">
        <button class="back-home" onclick="location.reload()">🏠 القائمة الرئيسية</button>
        <div class="header"><h3>🏆 شطرنج أحمد المحترف 🏆</h3><div id="my-id" style="font-size:0.7em;">جاري الاتصال...</div></div>
        <div id="board"></div>
        <div style="width:100%; max-width:400px; margin-top:15px;">
            <input type="text" id="friendId" placeholder="ID الخصم">
            <button onclick="alert('طلب التحدي قيد التطوير')">إرسال طلب تحدي</button>
        </div>
    </div>

    <div id="mafia-section" class="hidden">
        <button class="back-home" onclick="location.reload()">🏠 القائمة الرئيسية</button>
        <div class="header"><h3>🕵️‍♂️ نظام المافيا المطور</h3></div>
        <div id="mafia-setup" style="width:100%; max-width:400px;">
            <input type="text" id="mafiaPlayerName" placeholder="اسم اللاعب...">
            <button onclick="addMafiaPlayer()">إضافة لاعب</button>
            <div id="mafia-preview" style="margin:10px; color:#888;"></div>
            <button style="background:green;" onclick="startMafiaDist()">توزيع الأدوار + الراوي</button>
        </div>
        <div id="mafia-board" class="hidden" style="width:100%; max-width:400px;">
            <div id="narrator-name"></div>
            <div id="mafia-cards-container"></div>
        </div>
    </div>

    <div id="spy-section" class="hidden">
        <button class="back-home" onclick="location.reload()">🏠 القائمة الرئيسية</button>
        <div class="header"><h3>🔍 لعبة الجاسوس</h3></div>
        <div id="spy-setup" style="width:100%; max-width:400px;">
            <input type="text" id="spyPlayerName" placeholder="اسم اللاعب...">
            <button onclick="addSpyPlayer()">إضافة لاعب</button>
            <div id="spy-preview" style="margin:10px; color:#888;"></div>
            <button style="background:green;" onclick="startSpyDist()">توزيع الأماكن</button>
        </div>
        <div id="spy-board" class="hidden" style="width:100%; max-width:400px;">
            <div id="spy-cards-container"></div>
        </div>
    </div>

    <script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.js"></script>
    <script src="https://unpkg.com/peerjs@1.3.1/dist/peerjs.min.js"></script>

    <script>
        let isLocked = false;
        let mPlayers = [];
        let sPlayers = [];
        
        // قائمة الـ 50 كلمة للعبة الجاسوس
        const spyPlaces = [
            "مستشفى", "مدرسة", "غابة", "طائرة", "مطعم", "فندق", "سوق", "حديقة", "مطار", "محطة قطار", 
            "سفينة", "مخيم", "قلعة", "مسرح", "سينما", "ملعب كورة", "مكتبة", "متحف", "شاطئ", "بنك", 
            "مركز شرطة", "إطفائية", "جامعة", "مختبر", "فضاء", "تحت البحر", "منجم", "صحراء", "قطب متجمد", "مزرعة", 
            "قصر", "سجن", "ورشة", "صيدلية", "عيادة أسنان", "نادي رياضي", "صالون حلاقة", "مطعم شاورما", "منسف", "عرس", 
            "جنازة", "مصنع", "شركة", "قاعدة عسكرية", "برج إيفل", "الأهرامات", "مول", "سوبر ماركت", "محطة بنزين", "غرفة نوم"
        ];

        function showGame(type) {
            $('#main-menu').hide();
            if(type === 'chess') { $('#chess-section').removeClass('hidden'); initChess(); }
            else if(type === 'mafia') { $('#mafia-section').removeClass('hidden'); }
            else if(type === 'spy') { $('#spy-section').removeClass('hidden'); }
        }

        // --- شطرنج أحمد ---
        var board = null, game = new Chess(), peer = null;
        var pics = {
            wK: 'https://i.postimg.cc/m27HgYdg/king-black.jpg', wQ: 'https://i.postimg.cc/CxLfkvYW/813f791d-4209-474a-b3ae-32306e3d2632.jpg',
            wN: 'https://i.postimg.cc/d31K4Kx9/download.jpg', wB: 'https://i.postimg.cc/jjn9q4WQ/download.jpg',
            bP: 'https://i.postimg.cc/Nj9dgQFv/download.jpg', bK: 'https://i.postimg.cc/T19B8NZh/cbbe2978-b5df-45c8-ad43-05cfb6f57edb.jpg',
            bQ: 'https://i.postimg.cc/HsbtBvqn/ca232a37-372a-45be-b758-6e90cd5f70a1.jpg', bN: 'https://i.postimg.cc/DwFRgXMy/5f98a3d6-1c49-4903-a9c0-8a1b8a2de568.jpg',
            bB: 'https://i.postimg.cc/RVRx2dfm/download.jpg', wP: 'https://chessboardjs.com/img/chesspieces/wikipedia/wP.png'
        };

        function initChess() {
            let id = "Ahmed_" + Math.floor(1000 + Math.random() * 9000);
            peer = new Peer(id);
            peer.on('open', (i) => { 
                $('#my-id').text("ID: " + i); 
                board = Chessboard('board', {
                    draggable: true, position: 'start', 
                    pieceTheme: (p) => pics[p] || 'https://chessboardjs.com/img/chesspieces/wikipedia/' + p + '.png'
                }); 
            });
        }

        // --- مافيا (قانون أحمد) ---
        function addMafiaPlayer() {
            let n = $('#mafiaPlayerName').val().trim();
            if(n) { mPlayers.push(n); $('#mafia-preview').text("اللاعبين: " + mPlayers.join(" - ")); $('#mafiaPlayerName').val("").focus(); }
        }

        function startMafiaDist() {
            if(mPlayers.length < 4) return alert("لازم 4 لاعبين على الأقل!");
            let narratorIdx = Math.floor(Math.random() * mPlayers.length);
            let narratorName = mPlayers[narratorIdx];
            let actives = mPlayers.filter((_, i) => i !== narratorIdx);
            let roles = [];
            if (actives.length >= 10) { roles.push("مافيا القتل 🔪", "مافيا التسكيت 🤫"); } 
            else { roles.push("مافيا القتل 💀"); }
            roles.push("دكتور 💉");
            while(roles.length < actives.length) roles.push("مواطن 👨‍🌾");
            roles.sort(() => Math.random() - 0.5);

            $('#mafia-setup').hide();
            $('#mafia-board').removeClass('hidden');
            $('#narrator-name').html(`<div class="narrator-box">🎤 الراوي: ${narratorName}</div>`);
            
            let container = $('#mafia-cards-container');
            container.empty();
            actives.forEach((p, i) => {
                let card = $(`<div class="mafia-card" id="mc-${i}"><strong id="mpn-${i}">${p}</strong><div class="role-display" id="mrl-${i}">${roles[i]}</div></div>`);
                card.click(function() {
                    if(isLocked) return;
                    isLocked = true; $('.mafia-card').addClass('locked'); $(this).removeClass('locked');
                    $(`#mpn-${i}`).hide(); $(`#mrl-${i}`).fadeIn();
                    setTimeout(() => { 
                        $(`#mc-${i}`).addClass('fade-out'); 
                        setTimeout(() => { $(`#mc-${i}`).css('visibility', 'hidden'); isLocked = false; $('.mafia-card').removeClass('locked'); }, 500); 
                    }, 3000);
                });
                container.append(card);
            });
        }

        // --- الجاسوس (إضافة أحمد) ---
        function addSpyPlayer() {
            let n = $('#spyPlayerName').val().trim();
            if(n) { sPlayers.push(n); $('#spy-preview').text("اللاعبين: " + sPlayers.join(" - ")); $('#spyPlayerName').val("").focus(); }
        }

        function startSpyDist() {
            if(sPlayers.length < 3) return alert("لازم 3 لاعبين على الأقل!");
            let place = spyPlaces[Math.floor(Math.random() * spyPlaces.length)];
            let spyIdx = Math.floor(Math.random() * sPlayers.length);
            let roles = sPlayers.map((_, i) => (i === spyIdx ? "أنت الجاسوس 🕵️" : place));

            $('#spy-setup').hide();
            $('#spy-board').removeClass('hidden');
            let container = $('#spy-cards-container');
            container.empty();
            sPlayers.forEach((p, i) => {
                let card = $(`<div class="spy-card" id="sc-${i}"><strong id="spn-${i}">${p}</strong><div class="role-display" id="srl-${i}" style="color:#00ff00;">${roles[i]}</div></div>`);
                card.click(function() {
                    if(isLocked) return;
                    isLocked = true; $('.spy-card').addClass('locked'); $(this).removeClass('locked');
                    $(`#spn-${i}`).hide(); $(`#srl-${i}`).fadeIn();
                    setTimeout(() => { 
                        $(`#sc-${i}`).addClass('fade-out'); 
                        setTimeout(() => { $(`#sc-${i}`).css('visibility', 'hidden'); isLocked = false; $('.spy-card').removeClass('locked'); }, 500); 
                    }, 3000);
                });
                container.append(card);
            });
        }
    </script>
</body>
</html>
