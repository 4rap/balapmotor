<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>MotoGP Racer - Game Mengemudi Motor Pembalap</title>
    <style>
        * {
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }

        body {
            background: linear-gradient(145deg, #0a0f1e 0%, #0c1222 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Segoe UI', 'Poppins', 'Orbitron', monospace;
            margin: 0;
            padding: 20px;
        }

        .game-container {
            background: #0b0f1a;
            border-radius: 48px;
            padding: 20px 20px 25px;
            box-shadow: 0 25px 40px rgba(0, 0, 0, 0.6), inset 0 1px 1px rgba(255, 255, 255, 0.08);
            border: 1px solid rgba(255, 215, 0, 0.3);
        }

        canvas {
            display: block;
            margin: 0 auto;
            border-radius: 24px;
            box-shadow: 0 0 0 3px #1e2a3a, 0 15px 25px rgba(0, 0, 0, 0.5);
            cursor: pointer;
            background-color: #1a1f2c;
        }

        .info-panel {
            display: flex;
            justify-content: space-between;
            align-items: baseline;
            padding: 15px 20px 5px 20px;
            gap: 20px;
            flex-wrap: wrap;
        }

        .stats {
            background: #03060ecc;
            backdrop-filter: blur(6px);
            padding: 8px 18px;
            border-radius: 60px;
            font-weight: bold;
            font-family: 'Orbitron', monospace;
            letter-spacing: 1px;
            border: 1px solid #ffcc33;
            box-shadow: inset 0 1px 3px #00000055, 0 5px 8px #00000033;
        }

        .stats span:first-child {
            color: #ffaa33;
            text-shadow: 0 0 3px #ff8800;
        }

        .stats span:last-child {
            color: #f0f0f0;
            font-size: 1.5rem;
            font-weight: 800;
            margin-left: 10px;
            background: linear-gradient(135deg, #fff, #ffd966);
            -webkit-background-clip: text;
            background-clip: text;
            color: #ffd966;
        }

        .controls {
            display: flex;
            justify-content: center;
            gap: 35px;
            margin: 15px 0 10px 0;
        }

        .btn {
            background: radial-gradient(circle at 30% 10%, #2c2f3f, #10131f);
            border: none;
            font-size: 2.4rem;
            font-weight: bold;
            padding: 12px 28px;
            border-radius: 60px;
            color: #ffd966;
            font-family: 'Orbitron', monospace;
            cursor: pointer;
            box-shadow: 0 8px 0 #03060a;
            transition: 0.07s linear;
            text-transform: uppercase;
            letter-spacing: 3px;
            backdrop-filter: blur(4px);
            border: 1px solid #ffcc44aa;
        }

        .btn:active {
            transform: translateY(4px);
            box-shadow: 0 4px 0 #03060a;
        }

        .restart-btn {
            background: radial-gradient(circle at 30% 10%, #2a1f2c, #1a1020);
            color: #ff8c5a;
            border-color: #ff8450;
            box-shadow: 0 8px 0 #2a0a1a;
            font-size: 1.3rem;
            padding: 8px 24px;
        }

        .status {
            background: #000000aa;
            border-radius: 32px;
            padding: 5px 16px;
            font-size: 0.9rem;
            font-weight: bold;
            font-family: monospace;
            text-align: center;
            color: #ffcc88;
            backdrop-filter: blur(4px);
        }

        @media (max-width: 680px) {
            .btn { padding: 8px 20px; font-size: 2rem; }
            .stats span:last-child { font-size: 1.2rem; }
            .game-container { padding: 15px; }
        }
        .credit {
            text-align: center;
            font-size: 11px;
            color: #5f6a7a;
            margin-top: 12px;
            font-family: monospace;
        }
    </style>
</head>
<body>
<div>
    <div class="game-container">
        <canvas id="gameCanvas" width="600" height="600"></canvas>

        <div class="info-panel">
            <div class="stats">
                <span>🏁 SPEED</span>
                <span id="speedValue">0</span><span> km/h</span>
            </div>
            <div class="stats">
                <span>⭐ SCORE</span>
                <span id="scoreValue">0</span>
            </div>
            <div class="status" id="gameStatusText">
                🏍️ LET'S RACE 🏁
            </div>
        </div>

        <div class="controls">
            <button class="btn" id="leftBtn">◀ KIRI</button>
            <button class="btn restart-btn" id="restartBtn">🔄 RESTART</button>
            <button class="btn" id="rightBtn">KANAN ▶</button>
        </div>
        <div class="credit">
            ⚡ Tekan tombol KIRI / KANAN | Hindari motor lawan | Makin tinggi skor makin sulit!
        </div>
    </div>
</div>

<script>
    (function(){
        // ----- ELEMEN CANVAS -----
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // ----- PARAMETER GAME -----
        let gameRunning = true;
        let score = 0;
        let animationId = null;

        // ----- UKURAN OBJEK -----
        const PLAYER_WIDTH = 44;
        const PLAYER_HEIGHT = 32;
        const OBSTACLE_WIDTH = 42;
        const OBSTACLE_HEIGHT = 34;

        // ----- POSISI PLAYER -----
        let playerX = (canvas.width / 2) - (PLAYER_WIDTH / 2);
        const playerY = canvas.height - PLAYER_HEIGHT - 18;  // posisi bawah
        
        // ----- RINTANGAN (MOTOR LAWAN) -----
        let obstacles = [];
        
        // ----- SISTEM SPAWN -----
        let spawnCounter = 0;
        let currentSpawnDelay = 42;      // frame delay spawn (makin kecil makin sering)
        
        // ----- KECEPATAN DINAMIS (setiap frame bergerak) -----
        let currentGameSpeed = 2.8;       // pixel per frame
        
        // ----- FUNGSI UTAMA MENGHITUNG SPEED DAN SPAWN BERDASARKAN SKOR -----
        function updateDynamicDifficulty() {
            // Kecepatan maksimal 9.5, naik progresif
            let newSpeed = 2.6 + Math.floor(score / 280);
            if (newSpeed > 9.2) newSpeed = 9.2;
            if (newSpeed < 2.6) newSpeed = 2.6;
            currentGameSpeed = newSpeed;
            
            // Spawn delay: makin tinggi skor makin cepat spawn (semakin kecil delay)
            let rawDelay = 48 - Math.floor(score / 45);
            if (rawDelay < 20) rawDelay = 20;
            if (rawDelay > 48) rawDelay = 48;
            currentSpawnDelay = rawDelay;
        }
        
        // ----- RESET GAME FULL -----
        function resetGame() {
            gameRunning = true;
            score = 0;
            obstacles = [];
            // reset posisi player di tengah
            playerX = (canvas.width / 2) - (PLAYER_WIDTH / 2);
            // batas aman
            if(playerX < 0) playerX = 0;
            if(playerX + PLAYER_WIDTH > canvas.width) playerX = canvas.width - PLAYER_WIDTH;
            
            // spawn system reset
            spawnCounter = 8;   // biar langsung ada tantangan sedikit setelah reset
            updateDynamicDifficulty();   // speed awal = 2.8, spawn delay = 48
            currentGameSpeed = 2.6;
            updateDynamicDifficulty();   // hitung ulang berdasarkan score=0
            
            // Update tampilan UI
            document.getElementById('scoreValue').innerText = "0";
            document.getElementById('speedValue').innerText = Math.floor(currentGameSpeed * 22);
            document.getElementById('gameStatusText').innerHTML = "🏍️ RACING MODE 🏁";
            document.getElementById('gameStatusText').style.color = "#ffcc88";
        }
        
        // ----- SPAWN SATU RINTANGAN (MOTOR LAWAN) -----
        function spawnObstacle() {
            let randX = Math.random() * (canvas.width - OBSTACLE_WIDTH);
            // hindari spawn tepat di player terlalu dekat? tidak masalah karena y di atas
            obstacles.push({
                x: randX,
                y: -OBSTACLE_HEIGHT,
                width: OBSTACLE_WIDTH,
                height: OBSTACLE_HEIGHT
            });
        }
        
        // ----- UPDATE LOGIKA GAME (posisi, tabrakan, skor, spawn) -----
        function updateGame() {
            if (!gameRunning) return;
            
            // 1. update kecepatan dinamis berdasarkan score terbaru
            updateDynamicDifficulty();
            
            // 2. gerakkan semua rintangan ke bawah dengan kecepatan currentGameSpeed
            for (let i = 0; i < obstacles.length; i++) {
                obstacles[i].y += currentGameSpeed;
            }
            
            // 3. CEK TABRAKAN (sebelum hapus yg keluar biar akurat)
            for (let i = 0; i < obstacles.length; i++) {
                const obs = obstacles[i];
                if (playerX < obs.x + obs.width &&
                    playerX + PLAYER_WIDTH > obs.x &&
                    playerY < obs.y + obs.height &&
                    playerY + PLAYER_HEIGHT > obs.y) {
                    // TABRAKKAN !!
                    gameRunning = false;
                    document.getElementById('gameStatusText').innerHTML = "💥 CRASH! GAME OVER 💥";
                    document.getElementById('gameStatusText').style.color = "#ff6666";
                    return;  // stop proses update lebih lanjut (tidak ada penambahan skor, tidak spawn baru)
                }
            }
            
            // 4. HAPUS RINTANGAN YANG KELUAR BAWAH + TAMBAH SKOR
            let newObstacles = [];
            let scoreGain = 0;
            for (let i = 0; i < obstacles.length; i++) {
                const obs = obstacles[i];
                if (obs.y + obs.height > canvas.height) {
                    // rintangan berhasil dilewati tanpa tabrakan -> dapat poin
                    scoreGain += 10;
                } else {
                    newObstacles.push(obs);
                }
            }
            if (scoreGain > 0) {
                score += scoreGain;
                document.getElementById('scoreValue').innerText = score;
                // Update speed value display berdasarkan kecepatan baru
                let displaySpeed = Math.floor(currentGameSpeed * 21.5) + 40;
                if(displaySpeed > 320) displaySpeed = 320;
                document.getElementById('speedValue').innerText = displaySpeed;
            }
            obstacles = newObstacles;
            
            // 5. SISTEM SPAWN DENGAN COUNTER
            if (spawnCounter <= 0) {
                spawnObstacle();
                // set ulang spawn delay berdasarkan difficulty terbaru
                spawnCounter = currentSpawnDelay;
                // optional: kadang spawn ganda jika terlalu lama, tapi biar seru
                // sedikit random chance buat variasi (tapi tetap fair)
                if (Math.random() < 0.2 && score > 300 && currentSpawnDelay < 35) {
                    if(obstacles.length < 5) spawnObstacle(); // kadang spawn double untuk tantangan
                }
            } else {
                spawnCounter--;
            }
            
            // Update tampilan kecepatan di UI (setiap frame)
            let liveSpeedKMH = Math.floor(currentGameSpeed * 22.5) + 35;
            if(liveSpeedKMH > 335) liveSpeedKMH = 335;
            document.getElementById('speedValue').innerText = liveSpeedKMH;
            
            // Update status jika masih racing
            if(gameRunning){
                document.getElementById('gameStatusText').innerHTML = "🏁 RACING | SCORE: "+score+" 🏁";
            }
        }
        
        // ---------- GAMBAR SEMUA ELEMEN (MOTOGP STYLE) ----------
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // ----- ASPAL & EFISIENSI TRACK MOTOGP -----
            const asphaltGrd = ctx.createLinearGradient(0, 0, 0, canvas.height);
            asphaltGrd.addColorStop(0, "#171f2b");
            asphaltGrd.addColorStop(1, "#0e141f");
            ctx.fillStyle = asphaltGrd;
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // garis pinggir trek
            ctx.strokeStyle = "#ffcc44";
            ctx.lineWidth = 4;
            ctx.strokeRect(12, 12, canvas.width-24, canvas.height-24);
            ctx.beginPath();
            ctx.setLineDash([20, 35]);
            ctx.strokeStyle = "#ffdd88";
            ctx.lineWidth = 2.5;
            for(let i=0; i<canvas.width; i+=45){
                ctx.beginPath();
                ctx.moveTo(i, 0);
                ctx.lineTo(i, canvas.height);
                ctx.stroke();
            }
            ctx.setLineDash([]);
            
            // garis putih putus-putus di tengah (animasi gerak)
            let dashOffset = (performance.now() * 0.008) % 45;
            ctx.beginPath();
            ctx.setLineDash([25, 35]);
            ctx.lineWidth = 5;
            ctx.strokeStyle = "#f5f9ff";
            for(let y = dashOffset; y < canvas.height+40; y+=48){
                ctx.beginPath();
                ctx.moveTo(canvas.width/2, y);
                ctx.lineTo(canvas.width/2, y+28);
                ctx.stroke();
            }
            ctx.setLineDash([]);
            
            // bayangan aspal efek kecepatan
            ctx.fillStyle = "#ffffff0c";
            for(let i=0;i<12;i++){
                ctx.fillRect(30 + i*50, (performance.now()*0.5)%canvas.height, 20, 3);
            }
            
            // ----- GAMBAR MOTOR PEMBALAP (PLAYER) -----
            // bayangan motor
            ctx.shadowColor = "black";
            ctx.shadowBlur = 12;
            ctx.shadowOffsetX = 2;
            ctx.shadowOffsetY = 2;
            // body motor
            ctx.fillStyle = "#e03a3e";
            ctx.beginPath();
            ctx.roundRect(playerX, playerY, PLAYER_WIDTH, PLAYER_HEIGHT, 10);
            ctx.fill();
            ctx.fillStyle = "#cc2a2c";
            ctx.beginPath();
            ctx.roundRect(playerX+5, playerY-6, PLAYER_WIDTH-10, 10, 6);
            ctx.fill();
            // kaca helm & fairing
            ctx.fillStyle = "#2d2f36";
            ctx.fillRect(playerX+8, playerY+5, 8, 18);
            ctx.fillRect(playerX+PLAYER_WIDTH-16, playerY+5, 8, 18);
            ctx.fillStyle = "#ffd966";
            ctx.beginPath();
            ctx.arc(playerX+PLAYER_WIDTH/2, playerY-3, 8, 0, Math.PI*2);
            ctx.fill();
            ctx.fillStyle = "#111";
            ctx.beginPath();
            ctx.arc(playerX+PLAYER_WIDTH/2, playerY-4, 4, 0, Math.PI*2);
            ctx.fill();
            // roda
            ctx.fillStyle = "#1e1a15";
            ctx.fillRect(playerX+4, playerY+PLAYER_HEIGHT-6, 8, 8);
            ctx.fillRect(playerX+PLAYER_WIDTH-12, playerY+PLAYER_HEIGHT-6, 8, 8);
            ctx.fillStyle = "#999";
            ctx.fillRect(playerX+5, playerY+PLAYER_HEIGHT-5, 6, 4);
            ctx.fillRect(playerX+PLAYER_WIDTH-11, playerY+PLAYER_HEIGHT-5, 6, 4);
            
            // ----- GAMBAR RINTANGAN (MOTOR LAIN) -----
            for (let i = 0; i < obstacles.length; i++) {
                const o = obstacles[i];
                // efek gradien lawan
                let grad = ctx.createLinearGradient(o.x, o.y, o.x+15, o.y+OBSTACLE_HEIGHT);
                grad.addColorStop(0, "#9b2f3c");
                grad.addColorStop(1, "#5e1a24");
                ctx.fillStyle = grad;
                ctx.beginPath();
                ctx.roundRect(o.x, o.y, OBSTACLE_WIDTH, OBSTACLE_HEIGHT, 8);
                ctx.fill();
                ctx.fillStyle = "#dba842";
                ctx.fillRect(o.x+8, o.y-4, OBSTACLE_WIDTH-16, 8);
                ctx.fillStyle = "#2e2e2e";
                ctx.fillRect(o.x+5, o.y+5, 8, 18);
                ctx.fillRect(o.x+OBSTACLE_WIDTH-13, o.y+5, 8, 18);
                ctx.fillStyle = "#ff5533";
                ctx.beginPath();
                ctx.arc(o.x+OBSTACLE_WIDTH/2, o.y-2, 6, 0, Math.PI*2);
                ctx.fill();
                ctx.fillStyle = "#fff2bb";
                ctx.font = "bold 14px monospace";
                ctx.shadowBlur = 2;
                ctx.fillText("⚡", o.x+15, o.y+22);
            }
            
            // efek lampu belakang motor player
            ctx.fillStyle = "#ff8822";
            ctx.shadowBlur = 10;
            ctx.fillRect(playerX+PLAYER_WIDTH/2-3, playerY+PLAYER_HEIGHT-3, 6, 6);
            ctx.fillStyle = "#ff2200";
            ctx.fillRect(playerX+PLAYER_WIDTH/2-2, playerY+PLAYER_HEIGHT-2, 4, 4);
            
            ctx.shadowBlur = 0;
            // teks skill tambahan
            ctx.font = "bold 16monospace";
            ctx.fillStyle = "#ffcc77";
            ctx.shadowBlur = 2;
            ctx.fillText("MOTOGP", canvas.width-80, 32);
            if(!gameRunning){
                ctx.font = "900 36px 'Orbitron'";
                ctx.fillStyle = "#ff3333cc";
                ctx.shadowBlur = 8;
                ctx.fillText("CRASHED", canvas.width/2-95, canvas.height/2-40);
                ctx.font = "bold 18px monospace";
                ctx.fillStyle = "#f5aa44";
                ctx.fillText("TEKAN RESTART", canvas.width/2-80, canvas.height/2+30);
            }
        }
        
        // fungsi bantu roundRect
        if (!CanvasRenderingContext2D.prototype.roundRect) {
            CanvasRenderingContext2D.prototype.roundRect = function(x, y, w, h, r) {
                if (w < 2 * r) r = w / 2;
                if (h < 2 * r) r = h / 2;
                this.moveTo(x+r, y);
                this.lineTo(x+w-r, y);
                this.quadraticCurveTo(x+w, y, x+w, y+r);
                this.lineTo(x+w, y+h-r);
                this.quadraticCurveTo(x+w, y+h, x+w-r, y+h);
                this.lineTo(x+r, y+h);
                this.quadraticCurveTo(x, y+h, x, y+h-r);
                this.lineTo(x, y+r);
                this.quadraticCurveTo(x, y, x+r, y);
                return this;
            };
        }
        
        // ----- KONTROL TOMBOL (KLIK) -----
        const leftBtn = document.getElementById('leftBtn');
        const rightBtn = document.getElementById('rightBtn');
        const restartBtn = document.getElementById('restartBtn');
        
        // STEP per klik (pergerakan halus tapi responsif)
        const MOVE_STEP = 38;
        
        function moveLeft() {
            if(!gameRunning) return;
            let newX = playerX - MOVE_STEP;
            if(newX < 0) newX = 0;
            playerX = newX;
            // batas kanan juga dipastikan
            if(playerX + PLAYER_WIDTH > canvas.width) playerX = canvas.width - PLAYER_WIDTH;
        }
        
        function moveRight() {
            if(!gameRunning) return;
            let newX = playerX + MOVE_STEP;
            if(newX + PLAYER_WIDTH > canvas.width) newX = canvas.width - PLAYER_WIDTH;
            playerX = newX;
            if(playerX < 0) playerX = 0;
        }
        
        // Event klik & sentuh untuk tombol
        leftBtn.addEventListener('click', (e) => {
            e.preventDefault();
            moveLeft();
        });
        leftBtn.addEventListener('touchstart', (e) => {
            e.preventDefault();
            moveLeft();
        });
        
        rightBtn.addEventListener('click', (e) => {
            e.preventDefault();
            moveRight();
        });
        rightBtn.addEventListener('touchstart', (e) => {
            e.preventDefault();
            moveRight();
        });
        
        restartBtn.addEventListener('click', (e) => {
            e.preventDefault();
            resetGame();
            // memastikan tidak ada sisa kondisi crash
            gameRunning = true;
            // update UI tambahan
            document.getElementById('scoreValue').innerText = "0";
            document.getElementById('speedValue').innerText = "58";
            document.getElementById('gameStatusText').innerHTML = "🏍️ RACING MODE 🏁";
            document.getElementById('gameStatusText').style.color = "#ffcc88";
        });
        restartBtn.addEventListener('touchstart', (e) => {
            e.preventDefault();
            resetGame();
            gameRunning = true;
            document.getElementById('scoreValue').innerText = "0";
            document.getElementById('speedValue').innerText = "58";
            document.getElementById('gameStatusText').innerHTML = "🏍️ RACING MODE 🏁";
        });
        
        // ----- GAME LOOP (ANIMASI) -----
        function gameLoop() {
            if(gameRunning) {
                updateGame();   // update posisi, tabrakan, skor, spawn
            }
            draw();            // gambar semua elemen (walaupun game over)
            animationId = requestAnimationFrame(gameLoop);
        }
        
        // ----- KEYBOARD SUPPORT (tambahan) tapi tombol tetap utama -----
        window.addEventListener('keydown', (e) => {
            if(e.key === 'ArrowLeft') {
                e.preventDefault();
                moveLeft();
            } else if(e.key === 'ArrowRight') {
                e.preventDefault();
                moveRight();
            } else if(e.key === ' ' || e.key === 'Space' || e.key === 'r' || e.key === 'R') {
                if(e.key === 'r' || e.key === 'R') {
                    e.preventDefault();
                    resetGame();
                    gameRunning = true;
                    document.getElementById('gameStatusText').innerHTML = "🏍️ RACING MODE 🏁";
                }
            }
        });
        
        // ---- memulai game dengan reset awal ----
        resetGame();
        gameRunning = true;
        // pastikan nilai awal
        playerX = (canvas.width/2) - (PLAYER_WIDTH/2);
        // cek boundary
        if(playerX < 0) playerX = 0;
        if(playerX+PLAYER_WIDTH > canvas.width) playerX = canvas.width-PLAYER_WIDTH;
        obstacles = [];
        score = 0;
        spawnCounter = 6;
        updateDynamicDifficulty();
        currentGameSpeed = 2.8;
        document.getElementById('speedValue').innerText = "62";
        document.getElementById('scoreValue').innerText = "0";
        
        // jalankan loop
        gameLoop();
    })();
</script>
</body>
</html>
