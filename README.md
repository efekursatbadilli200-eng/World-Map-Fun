<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>WORLD MAP - Mobile Pixel Arena</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        body { background: #121212; color: #fff; overflow: hidden; height: 100vh; width: 100vw; display: flex; flex-direction: column; touch-action: none; }
        
        /* Üst Kontrol Barı */
        header { background: #1f1f1f; padding: 8px 12px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #333; z-index: 10; height: 45px; }
        .logo { font-size: 14px; font-weight: bold; color: #ffca28; }
        .canvas-select select { background: #2a2a2a; color: #fff; border: 1px solid #444; padding: 4px 8px; border-radius: 6px; font-size: 11px; outline: none; }

        /* Ana Oyun Alanı */
        #game-container { flex: 1; position: relative; overflow: hidden; background: #0a0a0a; touch-action: none; }
        #canvas-wrapper { position: absolute; transform-origin: 0 0; }
        canvas { display: block; image-rendering: pixelated; background: #ffffff; box-shadow: 0 0 20px rgba(0,0,0,0.8); }

        /* Arayüz Panelleri (UI) */
        .ui-panel { position: absolute; background: rgba(22, 22, 22, 0.9); backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px); border: 1px solid #333; border-radius: 10px; z-index: 5; }
        
        /* Sol Araç Çubuğu (Dikey) */
        #toolbar { top: 10px; left: 10px; display: flex; flex-direction: column; gap: 6px; padding: 6px; }
        .tool-btn { background: #2a2a2a; color: #fff; border: 1px solid #444; padding: 8px 10px; border-radius: 6px; cursor: pointer; text-align: center; font-size: 11px; white-space: nowrap; }
        .tool-btn.active { background: #ffca28; color: #000; font-weight: bold; border-color: #ffca28; }

        /* Sağ Alt Minimap & Koordinat */
        #minimap-container { bottom: 70px; right: 10px; padding: 4px; width: 90px; }
        #minimap-canvas { width: 82px; height: 82px; border: 1px solid #444; display: block; border-radius: 4px; }
        #coords { font-size: 9px; text-align: center; margin-top: 3px; color: #ffca28; font-family: monospace; }

        /* Alt Renk Paleti (Mobilde Yatay Kaydırılabilir) */
        #palette-wrapper { bottom: 10px; left: 10px; right: 10px; height: 50px; padding: 6px; display: flex; align-items: center; }
        #palette { display: flex; gap: 8px; overflow-x: auto; width: 100%; height: 100%; align-items: center; scrollbar-width: none; }
        #palette::-webkit-scrollbar { display: none; }
        .color-box { min-width: 32px; height: 32px; border-radius: 50%; cursor: pointer; border: 2px solid rgba(255,255,255,0.2); flex-shrink: 0; }
        .color-box.selected { border: 3px solid #fff; transform: scale(1.15); box-shadow: 0 0 8px rgba(255,255,255,0.5); }

        /* Sağ Chat Paneli (Açılır/Kapanır) */
        #chat-panel { top: 10px; right: 10px; width: 220px; height: 260px; display: flex; flex-direction: column; transition: transform 0.3s ease; }
        #chat-panel.collapsed { transform: translateX(240px); }
        .chat-tabs { display: flex; border-bottom: 1px solid #333; }
        .tab-btn { flex: 1; padding: 6px 2px; background: #1a1a1a; border: none; color: #888; cursor: pointer; font-size: 10px; }
        .tab-btn.active { background: #2a2a2a; color: #fff; font-weight: bold; }
        #chat-messages { flex: 1; padding: 6px; overflow-y: auto; font-size: 11px; display: flex; flex-direction: column; gap: 4px; }
        .msg { background: rgba(255,255,255,0.05); padding: 4px 6px; border-radius: 4px; word-break: break-word; }
        .chat-input-area { display: flex; padding: 4px; gap: 4px; border-top: 1px solid #333; }
        .chat-input-area input { flex: 1; background: #1a1a1a; border: 1px solid #333; color: #fff; padding: 5px; border-radius: 4px; outline: none; font-size: 10px; }
        .chat-input-area button { background: #ffca28; border: none; padding: 5px 8px; border-radius: 4px; cursor: pointer; font-weight: bold; font-size: 10px; }
        
        #toggle-chat { position: absolute; left: -32px; top: 0; background: rgba(22, 22, 22, 0.9); border: 1px solid #333; border-right: none; color: #fff; padding: 8px 6px; border-radius: 8px 0 0 8px; font-size: 12px; }

        /* Replay Çubuğu */
        #replay-bar { display: none; position: absolute; top: 55px; left: 50%; transform: translateX(-50%); padding: 8px 12px; width: 85%; max-width: 300px; text-align: center; }
        #replay-bar input { width: 100%; margin: 6px 0; }
    </style>
</head>
<body>

    <header>
        <div class="logo">🌍 WORLD MAP</div>
        <div class="canvas-select">
            <select id="canvasType">
                <option value="world">🗺️ Dünya Haritası</option>
                <option value="free">🎨 Serbest Tuval</option>
            </select>
        </div>
    </header>

    <div id="game-container">
        <div id="canvas-wrapper">
            <canvas id="mainCanvas" width="800" height="800"></canvas>
        </div>

        <!-- Araç Paneli -->
        <div id="toolbar" class="ui-panel">
            <button class="tool-btn active" id="btn-click">🎯 Tık</button>
            <button class="tool-btn" id="btn-pencil">✏️ Kalem</button>
            <button class="tool-btn" id="btn-eraser">🧹 Silgi</button>
            <button class="tool-btn" id="btn-replay">🎞️ Replay</button>
            <button class="tool-btn" id="btn-admin" style="background:#c62828;">🛡️ Admin</button>
        </div>

        <!-- Replay Çubuğu -->
        <div id="replay-bar" class="ui-panel">
            <div style="font-size: 11px;">🎞️ Zaman Çizelgesi</div>
            <input type="range" id="replaySlider" min="0" value="0" max="0">
            <button class="tool-btn" id="btn-close-replay">Çıkış</button>
        </div>

        <!-- Chat Paneli -->
        <div id="chat-panel" class="ui-panel collapsed">
            <button id="toggle-chat">💬</button>
            <div class="chat-tabs">
                <button class="tab-btn active" onclick="switchTab('tr')">🇹🇷 TR</button>
                <button class="tab-btn" onclick="switchTab('en')">🌐 EN</button>
                <button class="tab-btn" onclick="switchTab('dm')">🔒 DM</button>
            </div>
            <div id="chat-messages"></div>
            <div class="chat-input-area">
                <input type="text" id="chatInput" placeholder="Mesaj yazın...">
                <button onclick="sendMsg()">></button>
            </div>
        </div>

        <!-- Minimap & Koordinat -->
        <div id="minimap-container" class="ui-panel">
            <canvas id="minimap-canvas" width="800" height="800"></canvas>
            <div id="coords">X: 0 | Y: 0</div>
        </div>

        <!-- Alt Renk Paleti -->
        <div id="palette-wrapper" class="ui-panel">
            <div id="palette"></div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('mainCanvas');
        const ctx = canvas.getContext('2d');
        const miniCanvas = document.getElementById('minimap-canvas');
        const miniCtx = miniCanvas.getContext('2d');
        const wrapper = document.getElementById('canvas-wrapper');
        const container = document.getElementById('game-container');

        const CANVAS_SIZE = 800;
        let scale = 1;
        let pX = 0, pY = 0;
        let activeTool = 'click';
        let activeColor = '#000000';
        let history = [];
        let username = "Oyuncu_" + Math.floor(Math.random() * 8999 + 1000);

        // 24 Renk Paleti
        const colors = [
            '#000000','#ffffff','#7f7f7f','#c3c3c3','#880015','#b97a57',
            '#ed1c24','#ffaec9','#ff7f27','#ffc90e','#fff200','#efe4b0',
            '#22b14c','#b5e61d','#00a2e8','#99d9ea','#3f48cc','#7092be',
            '#a349a4','#c8bfe7','#464646','#d6a2e8','#ff3399','#00ffcc'
        ];

        // Paleti Doldur
        const paletteEl = document.getElementById('palette');
        colors.forEach((c, idx) => {
            const box = document.createElement('div');
            box.className = `color-box ${idx === 0 ? 'selected' : ''}`;
            box.style.background = c;
            box.onclick = () => {
                document.querySelectorAll('.color-box').forEach(b => b.classList.remove('selected'));
                box.classList.add('selected');
                activeColor = c;
            };
            paletteEl.appendChild(box);
        });

        // Tuval Beyaz Başlat
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);

        // Mobil Dokunmatik Dokunma & Kaydırma (Touch Eventler)
        let touchStartDist = 0;
        let touchStartPos = { x: 0, y: 0 };
        let isTouching = false;

        container.addEventListener('touchstart', (e) => {
            if (e.touches.length === 1) {
                isTouching = true;
                touchStartPos = { x: e.touches[0].clientX - pX, y: e.touches[0].clientY - pY };
                handleDraw(e.touches[0]);
            } else if (e.touches.length === 2) {
                touchStartDist = Math.hypot(
                    e.touches[0].clientX - e.touches[1].clientX,
                    e.touches[0].clientY - e.touches[1].clientY
                );
            }
        });

        container.addEventListener('touchmove', (e) => {
            if (e.touches.length === 1) {
                if (activeTool === 'pencil' || activeTool === 'eraser') {
                    handleDraw(e.touches[0]);
                } else {
                    pX = e.touches[0].clientX - touchStartPos.x;
                    pY = e.touches[0].clientY - touchStartPos.y;
                    updateTransform();
                }
            } else if (e.touches.length === 2) {
                const dist = Math.hypot(
                    e.touches[0].clientX - e.touches[1].clientX,
                    e.touches[0].clientY - e.touches[1].clientY
                );
                const zoom = dist / touchStartDist;
                if (scale * zoom >= 0.3 && scale * zoom <= 15) {
                    scale *= zoom;
                    touchStartDist = dist;
                    updateTransform();
                }
            }
        });

        container.addEventListener('touchend', () => { isTouching = false; });

        function updateTransform() {
            wrapper.style.transform = `translate(${pX}px, ${pY}px) scale(${scale})`;
        }

        function handleDraw(touch) {
            const rect = canvas.getBoundingClientRect();
            const x = Math.floor((touch.clientX - rect.left) / scale);
            const y = Math.floor((touch.clientY - rect.top) / scale);

            if (x >= 0 && x < CANVAS_SIZE && y >= 0 && y < CANVAS_SIZE) {
                document.getElementById('coords').innerText = `X: ${x} | Y: ${y}`;
                const color = activeTool === 'eraser' ? '#ffffff' : activeColor;
                ctx.fillStyle = color;
                ctx.fillRect(x, y, 1, 1);
                history.push({x, y, color});
                updateMinimap();
            }
        }

        function updateMinimap() {
            miniCtx.drawImage(canvas, 0, 0, CANVAS_SIZE, CANVAS_SIZE);
        }

        // Araç Butonları
        document.getElementById('btn-click').onclick = () => setTool('click');
        document.getElementById('btn-pencil').onclick = () => setTool('pencil');
        document.getElementById('btn-eraser').onclick = () => setTool('eraser');

        function setTool(tool) {
            activeTool = tool;
            document.querySelectorAll('.tool-btn').forEach(b => b.classList.remove('active'));
            document.getElementById(`btn-${tool}`).classList.add('active');
        }

        // Chat Paneli Aç/Kapa
        document.getElementById('toggle-chat').onclick = () => {
            document.getElementById('chat-panel').classList.toggle('collapsed');
        };

        // Replay Modu
        document.getElementById('btn-replay').onclick = () => {
            document.getElementById('replay-bar').style.display = 'block';
            const slider = document.getElementById('replaySlider');
            slider.max = history.length;
            slider.value = history.length;
        };

        document.getElementById('replaySlider').oninput = (e) => {
            const val = e.target.value;
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);
            for(let i = 0; i < val; i++) {
                const item = history[i];
                ctx.fillStyle = item.color;
                ctx.fillRect(item.x, item.y, 1, 1);
            }
            updateMinimap();
        };

        document.getElementById('btn-close-replay').onclick = () => {
            document.getElementById('replay-bar').style.display = 'none';
        };

        // Admin Paneli
        document.getElementById('btn-admin').onclick = () => {
            const pass = prompt("Admin Şifresi:");
            if (pass === "tac123") {
                const cmd = prompt("1: Haritayı Temizle\n2: Chati Temizle");
                if (cmd === "1") {
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);
                    history = [];
                    updateMinimap();
                    alert("Harita Sıfırlandı!");
                } else if (cmd === "2") {
                    document.getElementById('chat-messages').innerHTML = "";
                    alert("Chat Temizlendi!");
                }
            } else if (pass !== null) {
                alert("Yanlış Şifre!");
            }
        };

        function switchTab(tab) {
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            event.target.classList.add('active');
            document.getElementById('chat-messages').innerHTML = `<div class="msg"><i>${tab.toUpperCase()} kanalına geçildi.</i></div>`;
        }

        function sendMsg() {
            const input = document.getElementById('chatInput');
            if (!input.value.trim()) return;
            const msgBox = document.getElementById('chat-messages');
            const msgEl = document.createElement('div');
            msgEl.className = 'msg';
            msgEl.innerHTML = `<b>${username}:</b> ${input.value}`;
            msgBox.appendChild(msgEl);
            msgBox.scrollTop = msgBox.scrollHeight;
            input.value = "";
        }

        updateMinimap();
    </script>
</body>
</html>
# World-Map-Fun
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>DÜNYA HARİTASI - Mobile Pixel Arena</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        body { background: #121212; color: #fff; overflow: hidden; height: 100vh; width: 100vw; display: flex; flex-direction: column; touch-action: none; }
        
        /* Üst Kontrol Barı */
        header { background: #1f1f1f; padding: 8px 12px; display: flex; justify-content: space-between; align-items: center; z-index: 10; border-bottom: 1px solid #333; }
        .logo { font-size: 14px; font-weight: bold; color: #ffca28; }
        .stats { font-size: 11px; background: #2a2a2a; padding: 4px 8px; border-radius: 12px; border: 1px solid #444; display: flex; gap: 8px; align-items: center; }
        .mode-tag { background: #333; padding: 2px 6px; border-radius: 6px; font-size: 10px; color: #4caf50; }

        /* Oyun Tuvali */
        #canvas-wrapper { flex: 1; position: relative; overflow: hidden; background: #000; }
        canvas { display: block; image-rendering: pixelated; }

        /* Arayüz Panelleri */
        .ui-panel { position: absolute; z-index: 5; }
        
        /* Sol Araç Çubuğu */
        #tool-bar { top: 10px; left: 10px; display: flex; flex-direction: column; gap: 6px; }
        .btn { background: #2a2a2a; color: white; border: 1px solid #444; padding: 6px 10px; border-radius: 8px; font-size: 11px; font-weight: bold; cursor: pointer; }
        .btn:active { background: #444; }

        /* Sohbet Kutusu (Chat) */
        #chat-container { bottom: 60px; left: 10px; width: 220px; max-height: 160px; background: rgba(20, 20, 20, 0.85); border: 1px solid #444; border-radius: 8px; display: flex; flex-direction: column; padding: 6px; gap: 4px; z-index: 6; }
        #chat-messages { flex: 1; overflow-y: auto; font-size: 11px; display: flex; flex-direction: column; gap: 3px; max-height: 100px; color: #ddd; }
        .chat-msg { word-break: break-word; }
        .chat-msg b { color: #ffca28; }
        #chat-input-container { display: flex; gap: 4px; }
        #chat-input { flex: 1; background: #111; border: 1px solid #555; color: white; padding: 4px; border-radius: 4px; font-size: 11px; }
        #chat-send { background: #27ae60; border: none; color: white; padding: 4px 8px; border-radius: 4px; font-size: 11px; font-weight: bold; }

        /* Renk Paleti */
        #palette { bottom: 10px; left: 50%; transform: translateX(-50%); display: flex; gap: 5px; background: rgba(31, 31, 31, 0.9); padding: 6px; border-radius: 20px; border: 1px solid #444; }
        .color-box { width: 26px; height: 26px; border-radius: 50%; border: 2px solid transparent; cursor: pointer; }
        .color-box.selected { border-color: #fff; transform: scale(1.15); }
    </style>
</head>
<body>

    <header>
        <div class="logo">WORLD MAP</div>
        <div class="stats">
            <span id="pixel-count">Piksel: 10</span>
            <span id="timer">59s (+4)</span>
            <span id="void-timer">Void: 02:00:00</span>
        </div>
    </header>

    <div id="canvas-wrapper">
        <canvas id="gameCanvas"></canvas>

        <!-- Sol Üst Butonlar -->
        <div id="tool-bar" class="ui-panel">
            <button class="btn" id="undo-btn">↩ Geri Al</button>
            <button class="btn" id="check-void-btn">🎯 Void'i Kontrol Et</button>
        </div>

        <!-- Canlı Sohbet Kutusu -->
        <div id="chat-container">
            <div id="chat-messages">
                <div class="chat-msg"><b>Sistem:</b> Oyuna hoş geldin! Sohbetten mesaj yazabilirsin.</div>
            </div>
            <div id="chat-input-container">
                <input type="text" id="chat-input" placeholder="Mesaj yaz..." maxlength="50">
                <button id="chat-send">Gönder</button>
            </div>
        </div>

        <!-- Renk Paleti -->
        <div id="palette" class="ui-panel"></div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const pixelCountEl = document.getElementById('pixel-count');
        const timerEl = document.getElementById('timer');
        const voidTimerEl = document.getElementById('void-timer');
        const undoBtn = document.getElementById('undo-btn');
        const checkVoidBtn = document.getElementById('check-void-btn');
        const palette = document.getElementById('palette');

        const CANVAS_SIZE = 1000;
        canvas.width = CANVAS_SIZE;
        canvas.height = CANVAS_SIZE;

        let availablePixels = 10;
        let addAmount = 4; // Normalde 4 piksel
        let currentTimer = 59;
        
        // 2 Saatlik Void Zamanlayıcısı (7200 Saniye)
        let voidSecondsLeft = 7200; 
        let voidActive = false;
        let voidArea = { x: 450, y: 450, size: 100 }; // Harita ortasında 100x100 Void

        let history = [];
        let selectedColor = '#e74c3c';
        const colors = ['#e74c3c', '#e67e22', '#f1c40f', '#2ecc71', '#3498db', '#9b59b6', '#ffffff', '#000000'];

        // Tuval Beyaz Başlat
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);

        // Palet
        colors.forEach((color, idx) => {
            const box = document.createElement('div');
            box.className = `color-box ${idx === 0 ? 'selected' : ''}`;
            box.style.background = color;
            box.onclick = () => {
                document.querySelector('.color-box.selected')?.classList.remove('selected');
                box.classList.add('selected');
                selectedColor = color;
            };
            palette.appendChild(box);
        });

        // Void Doğurma Fonksiyonu
        function spawnVoid() {
            voidActive = true;
            ctx.fillStyle = '#111111'; // Void Rengi (Koyu Siyah/Mor)
            ctx.fillRect(voidArea.x, voidArea.y, voidArea.size, voidArea.size);
            addChatMessage("Sistem", "⚠️ YENİ VOID ALANI DOĞDU! Etrafını kapatın!");
        }

        // Piksel Koyma
        canvas.addEventListener('click', (e) => {
            if (availablePixels <= 0) return;

            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;

            const x = Math.floor((e.clientX - rect.left) * scaleX);
            const y = Math.floor((e.clientY - rect.top) * scaleY);

            const prevColorData = ctx.getImageData(x, y, 1, 1).data;
            const prevColor = `rgb(${prevColorData[0]}, ${prevColorData[1]}, ${prevColorData[2]})`;

            ctx.fillStyle = selectedColor;
            ctx.fillRect(x, y, 5, 5);

            history.push({ x, y, size: 5, color: prevColor });
            availablePixels--;
            updateUI();
        });

        // Geri Al
        undoBtn.onclick = () => {
            if (history.length === 0) return;
            const lastAction = history.pop();
            ctx.fillStyle = lastAction.color;
            ctx.fillRect(lastAction.x, lastAction.y, lastAction.size, lastAction.size);
            availablePixels++;
            updateUI();
        };

        // Void Etrafının Kapatıldığını Kontrol Etme
        checkVoidBtn.onclick = () => {
            if (!voidActive) {
                alert("Şu anda aktif bir Void yok veya zaten kazanıldı!");
                return;
            }
            
            // Basitleştirilmiş kontrol: Void etrafındaki çizgiler boyanmış mı?
            // Etrafı kapatıldıysa kazanılır ve yenilenme +1 olur:
            addAmount = 1;
            voidActive = false;
            addChatMessage("Sistem", "🎉 TEBRİKLER! Void alanının etrafı kapatıldı ve kazanıldı! Yenilenme hızı +1 piksel oldu.");
            alert("Void Kazandınız! Artık her 59 saniyede +1 piksel yenilenecek.");
        };

        // 59 Saniyelik Piksel Döngüsü
        setInterval(() => {
            currentTimer--;
            if (currentTimer <= 0) {
                availablePixels += addAmount;
                currentTimer = 59;
            }
            updateUI();
        }, 1000);

        // 2 Saatlik Void Sayacı
        setInterval(() => {
            if (voidSecondsLeft > 0) {
                voidSecondsLeft--;
                let hrs = Math.floor(voidSecondsLeft / 3600);
                let mins = Math.floor((voidSecondsLeft % 3600) / 60);
                let secs = voidSecondsLeft % 60;
                voidTimerEl.innerText = `Void: ${hrs.toString().padStart(2,'0')}:${mins.toString().padStart(2,'0')}:${secs.toString().padStart(2,'0')}`;
            } else {
                voidSecondsLeft = 7200; // Sayacı sıfırla
                spawnVoid();
            }
        }, 1000);

        function updateUI() {
            pixelCountEl.innerText = `Piksel: ${availablePixels}`;
            timerEl.innerText = `${currentTimer}s (+${addAmount})`;
        }

        // Chat (Sohbet) Mantığı
        const chatInput = document.getElementById('chat-input');
        const chatSend = document.getElementById('chat-send');
        const chatMessages = document.getElementById('chat-messages');

        function addChatMessage(user, msg) {
            const msgDiv = document.createElement('div');
            msgDiv.className = 'chat-msg';
            msgDiv.innerHTML = `<b>${user}:</b> ${msg}`;
            chatMessages.appendChild(msgDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        chatSend.onclick = () => {
            const text = chatInput.value.trim();
            if (text !== "") {
                addChatMessage("Oyuncu", text);
                chatInput.value = "";
            }
        };

        chatInput.addEventListener("keypress", (e) => {
            if (e.key === "Enter") chatSend.click();
        });
    </script>
</body>
</html>
