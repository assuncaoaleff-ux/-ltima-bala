<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Zumbi Survivor: Singularidade</title>
    <style>
        body { margin: 0; padding: 0; background-color: #1a1a1a; font-family: 'Arial Black', sans-serif; overflow: hidden; touch-action: none; }
        canvas { display: block; background-color: #1e272e; }
        
        #ui {
            position: absolute; top: 10px; width: 100%; display: flex; justify-content: space-around;
            align-items: center; color: white; font-size: 18px; z-index: 10; pointer-events: none;
            text-shadow: 2px 2px #000;
        }

        .top-btn {
            position: absolute; top: 50px; padding: 10px 15px; border-radius: 5px;
            font-weight: bold; z-index: 30; pointer-events: auto; cursor: pointer; border: 2px solid #fff;
        }
        #shop-btn { right: 20px; background: #f1c40f; color: #000; }
        #pause-btn { left: 20px; background: #3498db; color: #fff; }

        .overlay {
            display: none; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.95); border: 3px solid #f1c40f; padding: 15px;
            width: 85%; max-height: 70vh; overflow-y: auto; color: white; z-index: 40; text-align: center; border-radius: 15px;
        }

        #game-over-screen {
            display: none; position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.9); z-index: 100; flex-direction: column; justify-content: center; align-items: center;
        }
        #game-over-screen h1 { 
            color: #ff0000; font-size: 70px; margin: 0; text-shadow: 4px 4px #440000; 
            animation: tremer 0.2s infinite; 
        }

        @keyframes tremer { 
            0% { transform: translate(2px, 1px); } 
            50% { transform: translate(-1px, -2px); } 
            100% { transform: translate(1px, 2px); } 
        }

        .restart-btn { background: #fff; color: #000; border: none; padding: 15px 30px; font-size: 20px; font-weight: bold; border-radius: 10px; margin-top: 20px; cursor: pointer; }

        .shop-item { background: #333; margin: 8px 0; padding: 12px; border-radius: 10px; display: flex; justify-content: space-between; align-items: center; border: 1px solid #444; }
        .buy-btn { background: #27ae60; border: none; color: white; padding: 8px 12px; border-radius: 5px; font-weight: bold; }

        .controls { position: absolute; bottom: 20px; width: 100%; display: flex; justify-content: space-between; padding: 0 20px; box-sizing: border-box; z-index: 20; }
        .d-pad { display: grid; grid-template-columns: repeat(3, 60px); grid-template-rows: repeat(3, 60px); gap: 5px; }
        .btn { background: rgba(255, 255, 255, 0.2); border: 2px solid rgba(255, 255, 255, 0.5); border-radius: 10px; width: 60px; height: 60px; display: flex; align-items: center; justify-content: center; color: white; user-select: none; }
        .btn-fire { width: 90px; height: 90px; border-radius: 50%; background: rgba(231, 76, 60, 0.6); border-color: #e74c3c; align-self: flex-end; margin-bottom: 20px; }
    </style>
</head>
<body>

    <div id="ui">
        <div>R$ <span id="money">0</span></div>
        <div>Arma: <span id="weapon-name">Padrão</span></div>
    </div>

    <button id="pause-btn" class="top-btn" onclick="togglePause()">PAUSAR</button>
    <button id="shop-btn" class="top-btn" onclick="toggleShop()">LOJA</button>

    <div id="game-over-screen">
        <h1>GAME OVER</h1>
        <p style="color: white; font-size: 18px;">Dinheiro e Arma foram salvos!</p>
        <button class="restart-btn" onclick="location.reload()">RECOMECAR</button>
    </div>

    <div id="shop-menu" class="overlay">
        <h2 style="color: #f1c40f;">LOJA DE ELITE</h2>
        <div class="shop-item"><span>Pistola (50)</span><button class="buy-btn" onclick="buyWeapon('pistol', 50)">COMPRAR</button></div>
        <div class="shop-item"><span>Bomba (30)</span><button class="buy-btn" onclick="buyWeapon('bomb', 30)">COMPRAR</button></div>
        <div class="shop-item"><span>Metralhadora (100)</span><button class="buy-btn" onclick="buyWeapon('smg', 100)">COMPRAR</button></div>
        <div class="shop-item"><span>RPG (150)</span><button class="buy-btn" onclick="buyWeapon('rpg', 150)">COMPRAR</button></div>
        <div class="shop-item"><span>Rifle Laser (500)</span><button class="buy-btn" onclick="buyWeapon('laser', 500)">COMPRAR</button></div>
        <div class="shop-item"><span>Canhão Plasma (1200)</span><button class="buy-btn" onclick="buyWeapon('plasma', 1200)">COMPRAR</button></div>
        <div class="shop-item"><span>Minigun (3000)</span><button class="buy-btn" onclick="buyWeapon('minigun', 3000)">COMPRAR</button></div>
        <div class="shop-item"><span>Aniquilador (7000)</span><button class="buy-btn" onclick="buyWeapon('nuke', 7000)">COMPRAR</button></div>
        <div class="shop-item"><span>Singularidade Ômega (15000)</span><button class="buy-btn" onclick="buyWeapon('omega', 15000)">COMPRAR</button></div>
        <button style="margin-top:15px; background:#e74c3c; border:none; color:white; padding:12px; width:100%; border-radius:8px;" onclick="toggleShop()">FECHAR</button>
    </div>

    <div id="pause-menu" class="overlay" style="border-color: #3498db;">
        <h1 style="color: #3498db;">PAUSADO</h1>
        <button style="background:#3498db; color:white; border:none; padding:15px; width:100%; border-radius:8px;" onclick="togglePause()">RETOMAR</button>
    </div>

    <canvas id="gameCanvas"></canvas>

    <div class="controls">
        <div class="d-pad">
            <div style="grid-column: 2;" class="btn" id="up">▲</div>
            <div style="grid-column: 1; grid-row: 2;" class="btn" id="left">◀</div>
            <div style="grid-column: 3; grid-row: 2;" class="btn" id="right">▶</div>
            <div style="grid-column: 2; grid-row: 3;" class="btn" id="down">▼</div>
        </div>
        <div class="btn btn-fire" id="fire">TIRO</div>
    </div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const moneyElement = document.getElementById('money');
    const weaponNameElement = document.getElementById('weapon-name');
    const gameOverScreen = document.getElementById('game-over-screen');

    function resize() { canvas.width = window.innerWidth; canvas.height = window.innerHeight; }
    window.addEventListener('resize', resize);
    resize();

    // DADOS SALVOS
    let money = parseInt(localStorage.getItem('zumbi_money')) || 0;
    let savedWeaponId = localStorage.getItem('zumbi_weapon') || 'default';
    moneyElement.innerText = money;

    let score = 0, gameOver = false, isPaused = false, isShopOpen = false;
    let lastDir = { x: 1, y: 0 };
    const bullets = [], zombies = [], activeKeys = {};

    const weapons = {
        default: { name: "Padrão", speed: 10, size: 4, color: '#f1c40f', type: 'normal' },
        pistol: { name: "Pistola", speed: 18, size: 4, color: '#fff', type: 'normal' },
        smg: { name: "Metralhadora", speed: 22, size: 3, color: '#2ecc71', type: 'normal' },
        bomb: { name: "Bomba", speed: 6, size: 10, color: '#e74c3c', type: 'explosive' },
        rpg: { name: "RPG", speed: 12, size: 12, color: '#9b59b6', type: 'piercing' },
        laser: { name: "Rifle Laser", speed: 30, size: 2, color: '#00fbff', type: 'piercing' },
        plasma: { name: "Plasma", speed: 15, size: 20, color: '#a200ff', type: 'explosive' },
        minigun: { name: "Minigun", speed: 25, size: 4, color: '#ff9900', type: 'normal' },
        nuke: { name: "Aniquilador", speed: 8, size: 40, color: '#ff0000', type: 'explosive' },
        omega: { name: "Singularidade Ômega", speed: 40, size: 15, color: '#ffffff', type: 'piercing' }
    };

    // Equipa a arma salva logo no início
    let currentWeapon = weapons[savedWeaponId];
    weaponNameElement.innerText = currentWeapon.name;

    const player = { x: canvas.width/2, y: canvas.height/2, radius: 15, speed: 4 };

    function saveData() { 
        localStorage.setItem('zumbi_money', money); 
        // Salva o ID da arma atual para a próxima partida
        const weaponId = Object.keys(weapons).find(key => weapons[key].name === currentWeapon.name);
        localStorage.setItem('zumbi_weapon', weaponId);
    }

    function togglePause() { 
        if(gameOver) return;
        isPaused = !isPaused; 
        document.getElementById('pause-menu').style.display = isPaused ? 'block' : 'none'; 
    }

    function toggleShop() {
        if(gameOver) return;
        isShopOpen = !isShopOpen;
        isPaused = isShopOpen;
        document.getElementById('shop-menu').style.display = isShopOpen ? 'block' : 'none';
    }

    function buyWeapon(id, price) {
        if (money >= price) {
            money -= price;
            currentWeapon = weapons[id];
            weaponNameElement.innerText = currentWeapon.name;
            saveData(); // Salva dinheiro e arma nova imediatamente
            moneyElement.innerText = money;
            toggleShop();
        } else { alert("R$ insuficiente!"); }
    }

    function setupBtn(id, key) {
        const el = document.getElementById(id);
        el.addEventListener('touchstart', (e) => { e.preventDefault(); if(!isPaused && !gameOver) activeKeys[key] = true; });
        el.addEventListener('touchend', (e) => { e.preventDefault(); activeKeys[key] = false; });
    }
    setupBtn('up', 'u'); setupBtn('down', 'd'); setupBtn('left', 'l'); setupBtn('right', 'r');
    document.getElementById('fire').addEventListener('touchstart', (e) => { e.preventDefault(); if(!isPaused && !gameOver) shoot(); });

    function shoot() {
        bullets.push({ x: player.x, y: player.y, vx: lastDir.x * currentWeapon.speed, vy: lastDir.y * currentWeapon.speed, 
                       size: currentWeapon.size, color: currentWeapon.color, type: currentWeapon.type });
    }

    function spawn() {
        if (gameOver || isPaused) return;
        const x = Math.random() < 0.5 ? -30 : canvas.width + 30;
        zombies.push({ x: x, y: Math.random()*canvas.height, radius: 15, speed: 1.5 + (score/1000) });
    }
    setInterval(spawn, 1000);

    function update() {
        if (gameOver || isPaused) return;

        let dx = 0, dy = 0;
        if (activeKeys['u']) dy = -1; if (activeKeys['d']) dy = 1;
        if (activeKeys['l']) dx = -1; if (activeKeys['r']) dx = 1;
        if (dx !== 0 || dy !== 0) {
            player.x += dx * player.speed; player.y += dy * player.speed;
            lastDir = { x: dx, y: dy };
        }

        bullets.forEach((b, i) => {
            b.x += b.vx; b.y += b.vy;
            if (b.x < -100 || b.x > canvas.width + 100) bullets.splice(i, 1);
        });

        zombies.forEach((z, zi) => {
            const angle = Math.atan2(player.y - z.y, player.x - z.x);
            z.x += Math.cos(angle) * z.speed; z.y += Math.sin(angle) * z.speed;

            if (Math.hypot(player.x - z.x, player.y - z.y) < player.radius + z.radius) {
                gameOver = true; 
                saveData(); // Salva tudo ao morrer
                gameOverScreen.style.display = 'flex';
            }

            bullets.forEach((b, bi) => {
                if (Math.hypot(b.x - z.x, b.y - z.y) < z.radius + b.size) {
                    if (b.type === 'explosive') {
                        zombies.forEach((zn, zni) => { if(Math.hypot(b.x-zn.x, b.y-zn.y) < b.size*4) { zombies.splice(zni, 1); money+=5; score+=10; } });
                        bullets.splice(bi, 1);
                    } else if (b.type === 'piercing') {
                        zombies.splice(zi, 1); money+=5; score+=10;
                    } else {
                        zombies.splice(zi, 1); bullets.splice(bi, 1); money+=5; score+=10;
                    }
                    moneyElement.innerText = money; 
                    saveData();
                }
            });
        });
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.beginPath(); ctx.arc(player.x, player.y, player.radius, 0, Math.PI*2);
        ctx.fillStyle = '#3498db'; ctx.fill();

        bullets.forEach(b => {
            ctx.shadowBlur = currentWeapon.speed > 25 ? 15 : 0; ctx.shadowColor = b.color;
            ctx.beginPath(); ctx.arc(b.x, b.y, b.size, 0, Math.PI*2);
            ctx.fillStyle = b.color; ctx.fill();
        });
        ctx.shadowBlur = 0;

        zombies.forEach(z => {
            ctx.beginPath(); ctx.arc(z.x, z.y, z.radius, 0, Math.PI*2);
            ctx.fillStyle = '#27ae60'; ctx.fill();
        });

        update();
        requestAnimationFrame(draw);
    }
    draw();
</script>
</body>
</html>
