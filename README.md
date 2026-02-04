# Dash-finally-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Seattle Dash - Giselle ❤️</title>
    <style>
        /* Performance Optimization & Reset */
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: none; }
        body, html { width: 100%; height: 100%; overflow: hidden; background: #192a56; font-family: -apple-system, sans-serif; position: fixed; }

        #game-container { position: relative; width: 100vw; height: 100vh; background: #2f3640; overflow: hidden; }

        /* Smooth Moving Road Lines */
        .road-container { position: absolute; inset: 0; z-index: 1; }
        .lane-line { 
            position: absolute; top: -100%; bottom: 0; width: 4px; 
            background: repeating-linear-gradient(to bottom, transparent, transparent 40px, rgba(255,255,255,0.2) 40px, rgba(255,255,255,0.2) 80px);
            animation: roadMove 1s linear infinite;
        }
        @keyframes roadMove { from { transform: translateY(0); } to { transform: translateY(80px); } }

        /* Player - Optimized Transition */
        #player {
            position: absolute; bottom: 18%; width: 70px; height: 70px;
            font-size: 55px; display: flex; justify-content: center; align-items: center;
            transition: left 0.15s cubic-bezier(0.25, 0.46, 0.45, 0.94);
            z-index: 100; transform: translateX(-50%);
            will-change: left;
        }
        .running { animation: runBounce 0.25s infinite alternate ease-in-out; }
        @keyframes runBounce { from { margin-bottom: 0; } to { margin-bottom: 12px; } }

        /* HUD - Notch Safety */
        #hud {
            position: absolute; top: 12%; width: 100%;
            display: flex; flex-direction: column; align-items: center;
            z-index: 200; color: white; font-weight: bold; text-shadow: 1px 1px 4px #000;
        }
        .progress-wrap { width: 75%; height: 10px; background: rgba(0,0,0,0.5); border-radius: 5px; margin-top: 10px; overflow: hidden; border: 1px solid rgba(255,255,255,0.5); }
        #progress-bar { width: 0%; height: 100%; background: #00d2d3; transition: width 0.3s; }

        /* Entities & Particles */
        .item { position: absolute; font-size: 45px; transform: translateX(-50%); z-index: 80; will-change: top; }
        .heart-trail { 
            position: absolute; font-size: 24px; pointer-events: none; z-index: 70;
            animation: heartFadeOut 0.7s ease-out forwards; transform: translateX(-50%);
        }
        @keyframes heartFadeOut { 0% { opacity: 1; top: 0; } 100% { opacity: 0; top: -100px; } }

        /* Overlays */
        .overlay {
            position: fixed; inset: 0; background: rgba(0,0,0,0.95);
            display: flex; flex-direction: column; justify-content: center;
            align-items: center; text-align: center; z-index: 1000; padding: 40px; color: white;
        }
        .btn-start { padding: 20px 50px; border-radius: 50px; background: #00d2d3; color: white; border: none; font-size: 1.3rem; font-weight: bold; margin-top: 30px; box-shadow: 0 5px 15px rgba(0,0,0,0.3); }
    </style>
</head>
<body>

<div id="game-container">
    <div class="road-container">
        <div class="lane-line" style="left: 33.3%;"></div>
        <div class="lane-line" style="left: 66.6%;"></div>
    </div>
    <div id="player" class="running">🏃‍♀️</div>

    <div id="hud">
        <div>Tickets to Seattle: <span id="score">0</span>/15</div>
        <div class="progress-wrap"><div id="progress-bar"></div></div>
    </div>
</div>

<div id="start-menu" class="overlay">
    <h1 style="color: #00d2d3; font-size: 2.5rem;">Seattle Dash! 🏔️</h1>
    <p style="margin-top: 15px; font-size: 1.1rem; line-height: 1.4;">Swipe to dodge the study books and get to Seattle!</p>
    <button class="btn-start" onclick="startGame()">LET'S GO 🎶</button>
</div>

<div id="win-menu" class="overlay" style="display:none; background: white; color: #333;">
    <h1 style="color: #00d2d3;">Vacation Time! ✈️</h1>
    <p style="margin: 20px 0; font-size: 1.2rem;">You made it to Seattle! Check-in at citizenM is ready.</p>
    <p style="font-size: 1.8rem; font-weight: bold;">I love you, Giselle! ❤️</p>
</div>

<audio id="bg-audio" loop preload="auto">
    <source src="https://www.bensound.com/bensound-music/bensound-sunny.mp3" type="audio/mpeg">
</audio>

<script>
    let gameActive = false, tickets = 0, currentLane = 1, invincible = false;
    const laneX = [16.6, 50, 83.3];
    const player = document.getElementById('player');
    const scoreText = document.getElementById('score');
    const progressBar = document.getElementById('progress-bar');
    const music = document.getElementById('bg-audio');

    function startGame() {
        document.getElementById('start-menu').style.display = 'none';
        gameActive = true;
        music.play().catch(() => console.log("Audio play needs interaction."));
        spawnLoop();
    }

    // High-Precision Swipe Detection
    let touchStartX = 0;
    document.addEventListener('touchstart', (e) => touchStartX = e.touches[0].clientX, {passive: true});
    document.addEventListener('touchend', (e) => {
        if (!gameActive) return;
        const touchEndX = e.changedTouches[0].clientX;
        const diff = touchEndX - touchStartX;
        
        if (Math.abs(diff) > 35) { // Threshold to prevent accidental lane jumps
            if (diff > 0 && currentLane < 2) currentLane++;
            else if (diff < 0 && currentLane > 0) currentLane--;
            player.style.left = laneX[currentLane] + '%';
        }
    });

    function spawnLoop() {
        if (!gameActive) return;
        createObject();
        let speedFactor = Math.min(tickets * 20, 300);
        setTimeout(spawnLoop, 1000 - speedFactor);
    }

    function createObject() {
        const item = document.createElement('div');
        item.className = 'item';
        const typeRoll = Math.random();
        
        // Item types: k=key (power), t=ticket, b=book
        item.dataset.type = (typeRoll > 0.94) ? 'k' : (typeRoll > 0.55 ? 't' : 'b');
        item.innerHTML = (item.dataset.type === 'k') ? '🔑' : (item.dataset.type === 't' ? '✈️' : '📚');
        item.style.left = laneX[Math.floor(Math.random() * 3)] + '%';
        item.style.top = '-60px';
        document.getElementById('game-container').appendChild(item);

        let yPos = -60;
        const fallSpeed = invincible ? 14 : (8 + (tickets * 0.2));

        function move() {
            if (!gameActive) return;
            yPos += fallSpeed;
            item.style.top = yPos + 'px';

            const pRect = player.getBoundingClientRect();
            const iRect = item.getBoundingClientRect();

            // Refined collision logic
            if (pRect.left < iRect.right - 10 && pRect.right > iRect.left + 10 && 
                pRect.top < iRect.bottom - 10 && pRect.bottom > iRect.top + 10) {
                
                if (item.dataset.type === 'b' && !invincible) {
                    gameOver();
                    item.remove();
                    return;
                } else if (item.dataset.type === 't') {
                    collectTicket();
                } else if (item.dataset.type === 'k') {
                    powerUp();
                }
                item.remove();
                return;
            }

            if (yPos > window.innerHeight) {
                item.remove();
            } else {
                requestAnimationFrame(move);
            }
        }
        requestAnimationFrame(move);
    }

    function collectTicket() {
        tickets++;
        scoreText.innerText = tickets;
        progressBar.style.width = (tickets / 15 * 100) + '%';
        if (tickets >= 15) {
            gameActive = false;
            document.getElementById('win-menu').style.display = 'flex';
        }
    }

    function powerUp() {
        if (invincible) return;
        invincible = true;
        player.innerHTML = '✈️';
        player.classList.remove('running');
        
        const trailEffect = setInterval(() => {
            if (!invincible) { clearInterval(trailEffect); return; }
            const h = document.createElement('div');
            h.className = 'heart-trail';
            h.innerHTML = '❤️';
            h.style.left = player.style.left;
            h.style.top = (player.offsetTop + 40) + 'px';
            document.getElementById('game-container').appendChild(h);
            setTimeout(() => h.remove(), 700);
        }, 120);

        setTimeout(() => {
            invincible = false;
            player.innerHTML = '🏃‍♀️';
            player.classList.add('running');
        }, 4500);
    }

    function gameOver() {
        gameActive = false;
        alert("The Master's books caught up! Let's try again.");
        location.reload();
    }

    player.style.left = '50%';
</script>
</body>
</html>
