minecraft game.

    let score = 0; 

    let highscore = localStorage.getItem('mc_highscore') || 0; 

    let mouseX = 0; 

    let mouseY = 0; 

    let gameActive = false; 

    let difficulty = 1;  

    let activeIntervals = []; 

  

    document.addEventListener('mousemove', (e) => { mouseX = e.clientX; mouseY = e.clientY; }); 

  

    // --- UI ELEMENT (Säkert skapade) --- 

    const scoreboard = document.createElement('div'); 

    scoreboard.style.cssText = 'position:fixed; top:20px; right:20px; font-family:monospace; font-size:20px; color:white; background:rgba(0,0,0,0.8); padding:10px; border:4px solid #555; z-index:10005; pointer-events:none; display:none;'; 

    document.body.appendChild(scoreboard); 

  

    const menu = document.createElement('div'); 

    menu.style.cssText = 'position:fixed; top:50%; left:50%; transform:translate(-50%, -50%); background:#333; border:8px solid #000; padding:30px; color:white; font-family:monospace; text-align:center; z-index:10010; box-shadow: 0 0 100px rgba(0,0,0,0.8); min-width: 300px; display:block;'; 

    document.body.appendChild(menu); 

  

    function clearMenu() { 

        while (menu.firstChild) { 

            menu.removeChild(menu.firstChild); 

        } 

    } 

  

    function showMenu(type) { 

        gameActive = false; 

        activeIntervals.forEach(clearInterval); 

        activeIntervals = []; 

        document.querySelectorAll('.game-obj').forEach(obj => obj.remove()); 

        scoreboard.style.display = 'none'; 

        menu.style.display = 'block'; 

        clearMenu(); 

  

        const title = document.createElement('h1'); 

        title.textContent = type === 'start' ? 'MINECRAFT CLICKER' : 'GAME OVER'; 

        title.style.color = type === 'start' ? '#5d994d' : '#ff4d4d'; 

        menu.appendChild(title); 

  

        if (type === 'over') { 

            const stats = document.createElement('p'); 

            stats.textContent = `POÄNG: ${score} | BÄSTA: ${highscore}`; 

            menu.appendChild(stats); 

            if (score >= highscore && score > 0) { 

                highscore = score; 

                localStorage.setItem('mc_highscore', highscore); 

                const newRec = document.createElement('p'); 

                newRec.textContent = "NYTT REKORD!"; 

                newRec.style.color = "gold"; 

                menu.appendChild(newRec); 

            } 

        } 

  

        const btnStyle = 'display:block; width:100%; margin:10px auto; padding:12px; background:#555; border:4px solid #000; color:white; cursor:pointer; font-family:monospace; font-size:18px;'; 

         

        const playBtn = document.createElement('button'); 

        playBtn.textContent = type === 'start' ? 'STARTA' : 'SPELA IGEN'; 

        playBtn.style.cssText = btnStyle; 

        playBtn.onclick = startGame; 

        menu.appendChild(playBtn); 

  

        const setBtn = document.createElement('button'); 

        setBtn.textContent = 'INSTÄLLNINGAR'; 

        setBtn.style.cssText = btnStyle; 

        setBtn.onclick = showSettings; 

        menu.appendChild(setBtn); 

    } 

  

    function showSettings() { 

        clearMenu(); 

        const title = document.createElement('h1'); 

        title.textContent = "INSTÄLLNINGAR"; 

        menu.appendChild(title); 

  

        const label = document.createElement('p'); 

        label.textContent = `SVÅRHET: ${difficulty}x`; 

        menu.appendChild(label); 

  

        const input = document.createElement('input'); 

        input.type = 'range'; input.min = '0.5'; input.max = '3'; input.step = '0.5'; 

        input.value = difficulty; 

        input.style.width = '100%'; 

        input.oninput = () => { difficulty = input.value; label.textContent = `SVÅRHET: ${difficulty}x`; }; 

        menu.appendChild(input); 

  

        const backBtn = document.createElement('button'); 

        backBtn.textContent = 'KLAR'; 

        backBtn.style.cssText = 'display:block; width:100%; margin-top:20px; padding:10px; background:#5d994d; border:4px solid #000; color:white; cursor:pointer; font-family:monospace;'; 

        backBtn.onclick = () => showMenu('start'); 

        menu.appendChild(backBtn); 

    } 

  

    function explode(x, y) { 

        const blast = document.createElement('div'); 

        blast.className = 'game-obj'; 

        blast.style.cssText = `position:fixed; left:${x-75}px; top:${y-75}px; width:150px; height:150px; background:orange; border-radius:50%; opacity:0.6; z-index:10001; pointer-events:none; border:4px solid red;`; 

        document.body.appendChild(blast); 

        setTimeout(() => blast.remove(), 400); 

  

        if (Math.hypot(mouseX - x, mouseY - y) < 90) { 

            showMenu('over'); 

        } 

    } 

  

    function spawnCreeper() { 

        if (!gameActive) return; 

        const size = 60; 

        const x = 50 + Math.random() * (window.innerWidth - 100 - size); 

        let y = -size; 

  

        const creeper = document.createElement('div'); 

        creeper.className = 'game-obj'; 

        creeper.style.cssText = `position:fixed; left:${x}px; top:${y}px; width:${size}px; height:${size}px; background:#49aa3d; border:3px solid #000; z-index:9998; cursor:pointer;`; 

         

        const face = document.createElement('div'); 

        face.style.cssText = 'width:100%; height:100%; background:black; clip-path:polygon(20% 20%, 40% 20%, 40% 40%, 60% 40%, 60% 20%, 80% 20%, 80% 50%, 70% 50%, 70% 90%, 60% 90%, 60% 60%, 40% 60%, 40% 90%, 30% 90%, 30% 50%, 20% 50%); pointer-events:none;'; 

        creeper.appendChild(face); 

  

        creeper.onclick = () => { score += 20; scoreboard.textContent = `POÄNG: ${score}`; creeper.remove(); }; 

        document.body.appendChild(creeper); 

  

        const fall = setInterval(() => { 

            if (!gameActive) { clearInterval(fall); return; } 

            y += (4 * difficulty); 

            creeper.style.top = y + 'px'; 

            if (y > window.innerHeight - size) { 

                clearInterval(fall); 

                if (creeper.parentNode) { explode(x + (size/2), y + (size/2)); creeper.remove(); } 

            } 

        }, 20); 

    } 

  

    function spawnBlock() { 

        if (!gameActive) return; 

        const size = 60; 

        const x = 50 + Math.random() * (window.innerWidth - 100 - size); 

        const y = 50 + Math.random() * (window.innerHeight - 100 - size); 

        const isTNT = Math.random() < (0.15 * difficulty); 

  

        const block = document.createElement('div'); 

        block.className = 'game-obj'; 

        block.style.cssText = `position:fixed; left:${x}px; top:${y}px; width:${size}px; height:${size}px; z-index:9997; cursor:pointer; border:3px solid #000; display:flex; align-items:center; justify-content:center;`; 

         

        if(isTNT) { 

            block.style.background = '#db2f2f'; 

            const tntText = document.createElement('span'); 

            tntText.textContent = "TNT"; tntText.style.color = "white"; tntText.style.fontWeight = "bold"; 

            block.appendChild(tntText); 

            block.onclick = () => explode(x + (size/2), y + (size/2)); 

        } else { 

            block.style.background = '#5d994d'; 

            block.style.borderTop = '12px solid #79b463'; 

            block.style.boxShadow = 'inset 0 -15px 0 #795548'; 

            block.onclick = () => { score += 10; scoreboard.textContent = `POÄNG: ${score}`; block.remove(); }; 

        } 

        document.body.appendChild(block); 

        setTimeout(() => { if(block.parentNode) block.remove(); }, 3000 / difficulty); 

    } 

  

    function startGame() { 

        score = 0; 

        gameActive = true; 

        menu.style.display = 'none'; 

        scoreboard.style.display = 'block'; 

        scoreboard.textContent = 'POÄNG: 0'; 

        activeIntervals.push(setInterval(spawnBlock, 1200 / difficulty)); 

        activeIntervals.push(setInterval(spawnCreeper, 3000 / difficulty)); 

    } 

  

    showMenu('start'); 

})(); 

 

 
