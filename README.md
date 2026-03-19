<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Lendário Beat Clash</title>
    <style>
        :root {
            --purple: #a155ff; --blue: #00ffff; --green: #00ff00; --red: #ff0000;
            --gold: #ffd700; --dark: #0d0d0d;
        }

        body {
            background: var(--dark);
            color: white;
            font-family: 'Segoe UI', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            height: 100vh;
            overflow: hidden;
            touch-action: manipulation;
        }

        h1 { color: var(--gold); margin: 10px 0; text-transform: uppercase; font-size: 1.4rem; text-shadow: 2px 2px #ff00ff; }
        #stats { font-size: 1.1rem; margin-bottom: 5px; visibility: hidden; }

        #game-container {
            position: relative;
            width: 100%;
            max-width: 400px;
            height: 70vh;
            background: linear-gradient(180deg, #1a1a1a 0%, #000 100%);
            border: 3px solid var(--gold);
            border-radius: 15px;
            overflow: hidden;
        }

        .overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.95);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 100;
            text-align: center;
            padding: 20px;
            box-sizing: border-box;
        }

        #game-over-screen { display: none; }

        .menu-title { font-size: 2.2rem; color: var(--gold); margin: 0; text-shadow: 0 0 15px rgba(255, 215, 0, 0.5); }
        .menu-creator { font-size: 0.9rem; color: #aaa; margin: 5px 0 15px 0; letter-spacing: 1px; }
        .music-credits { font-size: 0.75rem; color: #666; font-style: italic; margin-bottom: 25px; max-width: 80%; }

        .menu-btn {
            width: 220px; padding: 15px; margin: 8px;
            border: none; border-radius: 30px;
            font-size: 1.1rem; font-weight: bold; cursor: pointer;
            text-transform: uppercase; transition: 0.2s;
            box-shadow: 0 4px #444;
        }
        .btn-start, .btn-restart { background: var(--gold); color: black; }
        .btn-continue { background: var(--blue); color: black; }

        #health-wrap { width: 80%; height: 12px; background: var(--red); margin: 10px 0; border: 2px solid white; border-radius: 10px; overflow: hidden; visibility: hidden; }
        #health-bar { width: 50%; height: 100%; background: var(--green); float: right; transition: width 0.2s; }

        #receptor-row { display: flex; justify-content: space-around; padding-top: 25px; position: relative; z-index: 5; }
        .receptor { width: 45px; height: 45px; border: 3px solid #333; display: flex; justify-content: center; align-items: center; font-size: 1.3rem; border-radius: 10px; transition: 0.1s; }
        
        .note { position: absolute; width: 45px; height: 45px; display: flex; justify-content: center; align-items: center; font-size: 1.3rem; font-weight: bold; border-radius: 10px; }

        .active { transform: scale(1.15); background: white !important; color: black !important; }

        #mobile-controls { 
            display: none; 
            position: absolute;
            bottom: 20px;
            width: 100%;
            justify-content: space-around; 
            align-items: center; 
            z-index: 10;
        }
        .m-btn { 
            width: 60px; height: 60px; border-radius: 50%; 
            border: 2px solid rgba(255,255,255,0.3); 
            font-size: 1.5rem; color: white; cursor: pointer;
            box-shadow: 0 0 10px rgba(0,0,0,0.5);
        }

        footer { font-size: 0.7rem; color: #555; margin-top: auto; padding-bottom: 5px; }

        .k-0 { color: var(--purple); border: 3px solid var(--purple); } 
        .k-1 { color: var(--blue); border: 3px solid var(--blue); }
        .k-2 { color: var(--green); border: 3px solid var(--green); } 
        .k-3 { color: var(--red); border: 3px solid var(--red); }
    </style>
</head>
<body>

    <h1>Lendário Beat Clash</h1>
    <div id="health-wrap"><div id="health-bar"></div></div>
    <div id="stats">PONTOS: <span id="score" style="color:var(--gold)">0</span> | COMBO: <span id="combo">0</span></div>

    <div id="game-container">
        <div id="start-screen" class="overlay">
            <h2 class="menu-title">Lendário Beat Clash</h2>
            <p class="menu-creator">CRIADORA: LENDÁRIO STUDIO</p>
            <p class="music-credits">Música: Pinball Spring</p>
            <button class="menu-btn btn-start" onclick="startGame()">Jogar Agora</button>
        </div>

        <div id="game-over-screen" class="overlay">
            <h2 style="color:var(--red); font-size: 2.5rem; margin: 0;">DERROTA!</h2>
            <p id="final-score-text" style="margin: 10px 0 20px 0;"></p>
            <button class="menu-btn btn-restart" onclick="restartGame()">Tentar de Novo</button>
            <button class="menu-btn btn-continue" onclick="continueGame()">Continuar (-500 pts)</button>
        </div>

        <div id="receptor-row">
            <div class="receptor k-0" id="r-0">←</div>
            <div class="receptor k-1" id="r-1">↓</div>
            <div class="receptor k-2" id="r-2">↑</div>
            <div class="receptor k-3" id="r-3">→</div>
        </div>
        
        <div id="note-layer"></div>

        <div id="mobile-controls">
            <button class="m-btn" style="background:var(--purple)" onmousedown="handleInput(0)" ontouchstart="handleInput(0)">←</button>
            <button class="m-btn" style="background:var(--blue)" onmousedown="handleInput(1)" ontouchstart="handleInput(1)">↓</button>
            <button class="m-btn" style="background:var(--green)" onmousedown="handleInput(2)" ontouchstart="handleInput(2)">↑</button>
            <button class="m-btn" style="background:var(--red)" onmousedown="handleInput(3)" ontouchstart="handleInput(3)">→</button>
        </div>
    </div>

    <footer>© 2025 – 2026 Lendário Studio. Todos os direitos reservados.</footer>

    <script>
        const noteLayer = document.getElementById('note-layer');
        const scoreEl = document.getElementById('score');
        const comboEl = document.getElementById('combo');
        const healthBar = document.getElementById('health-bar');
        const gameOverScreen = document.getElementById('game-over-screen');
        const startScreen = document.getElementById('start-screen');
        const mobileControls = document.getElementById('mobile-controls');
        
        const gameMusic = new Audio('https://files.catbox.moe/es4e1h.mp3');
        gameMusic.loop = true;

        let score = 0;
        let combo = 0;
        let health = 50;
        let speed = 5; 
        let isPlaying = false;
        let spawnInterval;

        function updateUI() {
            scoreEl.innerText = score;
            comboEl.innerText = combo;
            healthBar.style.width = health + "%";
            if (health <= 0 && isPlaying) endGame();
        }

        function startGame() {
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
            mobileControls.style.display = 'flex';
            document.getElementById('stats').style.visibility = 'visible';
            document.getElementById('health-wrap').style.visibility = 'visible';
            gameMusic.play().catch(e => {});
            resetGameState();
        }

        function endGame() {
            isPlaying = false;
            gameMusic.pause();
            clearInterval(spawnInterval);
            document.getElementById('final-score-text').innerText = "Pontos Totais: " + score;
            mobileControls.style.display = 'none'; 
            gameOverScreen.style.display = 'flex';
        }

        function restartGame() {
            score = 0; combo = 0; health = 50;
            startGame();
        }

        function continueGame() {
            if (score >= 500) {
                score -= 500; health = 50;
                gameOverScreen.style.display = 'none';
                mobileControls.style.display = 'flex';
                gameMusic.play();
                isPlaying = true;
                updateUI();
                spawnInterval = setInterval(createNote, 800);
            } else { alert("Você precisa de 500 pontos!"); }
        }

        function resetGameState() {
            isPlaying = true;
            noteLayer.innerHTML = ''; 
            updateUI();
            clearInterval(spawnInterval);
            spawnInterval = setInterval(createNote, 800);
        }

        function createNote() {
            if (!isPlaying) return;
            const lane = Math.floor(Math.random() * 4);
            const note = document.createElement('div');
            note.className = `note k-${lane}`;
            note.style.left = [5, 30, 55, 80][lane] + '%';
            
            // LOGICA: NASCE EM BAIXO
            let containerHeight = document.getElementById('game-container').offsetHeight;
            let pos = containerHeight; 
            note.style.top = pos + 'px';
            
            note.innerText = ['←', '↓', '↑', '→'][lane];
            note.dataset.lane = lane;
            note.dataset.hit = "false";
            noteLayer.appendChild(note);

            const move = setInterval(() => {
                if (!isPlaying) { clearInterval(move); note.remove(); return; }
                
                pos -= speed; // SUBINDO
                note.style.top = pos + 'px';
                
                // Se passar do topo (ERROU)
                if (pos < -50) {
                    if (note.dataset.hit === "false") { 
                        health -= 10; combo = 0; updateUI(); 
                    }
                    clearInterval(move); note.remove();
                }
            }, 16);
        }

        function handleInput(laneIndex) {
            if (!isPlaying) return;
            const receptor = document.getElementById(`r-${laneIndex}`);
            receptor.classList.add('active');
            setTimeout(() => receptor.classList.remove('active'), 100);

            const notes = document.querySelectorAll('.note');
            notes.forEach(note => {
                const noteTop = note.offsetTop;
                // ACERTO NOS RECEPTORES (TOP)
                if (note.dataset.lane == laneIndex && noteTop > 10 && noteTop < 100 && note.dataset.hit === "false") {
                    note.dataset.hit = "true";
                    combo++; score += 100;
                    health = Math.min(100, health + 4);
                    note.remove(); updateUI();
                }
            });
        }

        window.addEventListener('keydown', (e) => {
            const keys = ['ArrowLeft', 'ArrowDown', 'ArrowUp', 'ArrowRight'];
            const idx = keys.indexOf(e.key);
            if (idx !== -1) { e.preventDefault(); handleInput(idx); }
        });
    </script>
</body>
</html>
