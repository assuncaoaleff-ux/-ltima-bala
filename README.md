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
    dr
