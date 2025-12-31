<!DOCTYPE html>
<html>
<head>
    <title>Neon-Strike by Priyankaa</title>
    <style>
        body { 
            background: #0d1117; 
            display: flex; 
            justify-content: center; 
            align-items: center; 
            height: 100vh; 
            margin: 0; 
            overflow: hidden;
            font-family: 'Segoe UI', sans-serif;
        }
        canvas { 
            background: radial-gradient(circle, #1a1a2e 0%, #000 100%); 
            border: 4px solid #30363d;
            box-shadow: 0 0 30px rgba(0, 210, 255, 0.2);
            border-radius: 12px;
            cursor: pointer;
        }
    </style>
</head>
<body>

<canvas id="gameCanvas" width="600" height="400"></canvas>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    // Game State
    let gameState = "START"; // START, PLAYING, GAMEOVER
    let score = 0;
    let lives = 3;
    let level = 1;

    // Ball & Paddle
    let ballRadius = 8;
    let x = canvas.width / 2;
    let y = canvas.height - 35;
    let dx = 4;
    let dy = -4;
    let paddleHeight = 12;
    let paddleWidth = 90;
    let paddleX = (canvas.width - paddleWidth) / 2;

    // Input
    let rightPressed = false;
    let leftPressed = false;

    // Bricks
    const brickRowCount = 4;
    const brickColumnCount = 7;
    const brickWidth = 70;
    const brickHeight = 20;
    const brickPadding = 10;
    const brickOffsetTop = 60;
    const brickOffsetLeft = 25;
    let bricks = [];
    const levelColors = ["#00d2ff", "#ff007f", "#33ff57", "#ffbd33", "#9d33ff"];

    function initBricks() {
        for(let c=0; c<brickColumnCount; c++) {
            bricks[c] = [];
            for(let r=0; r<brickRowCount; r++) {
                bricks[c][r] = { x: 0, y: 0, status: 1 };
            }
        }
    }
    initBricks();

    // Audio Engine
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playSound(freq, type) {
        if(audioCtx.state === 'suspended') audioCtx.resume();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = type;
        osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
        gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.1);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + 0.1);
    }

    document.addEventListener("keydown", (e) => {
        if(e.key == "Right" || e.key == "ArrowRight") rightPressed = true;
        else if(e.key == "Left" || e.key == "ArrowLeft") leftPressed = true;
    });
    document.addEventListener("keyup", (e) => {
        if(e.key == "Right" || e.key == "ArrowRight") rightPressed = false;
        else if(e.key == "Left" || e.key == "ArrowLeft") leftPressed = false;
    });

    canvas.addEventListener('click', () => {
        if(gameState === "START" || gameState === "GAMEOVER") {
            gameState = "PLAYING";
            lives = 3;
            level = 1;
            score = 0;
            dx = 4; dy = -4;
            initBricks();
        }
    });

    function drawStartScreen() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        // Title
        ctx.fillStyle = "#00d2ff";
        ctx.font = "bold 40px Segoe UI";
        ctx.textAlign = "center";
        ctx.fillText("NEON-STRIKE", canvas.width/2, 160);
        
        // Credits
        ctx.fillStyle = "#ffffff";
        ctx.font = "18px Segoe UI";
        ctx.fillText("Developed by PRIYANKAA", canvas.width/2, 200);
        
        // Pulse Effect for Click Text
        const pulse = Math.sin(Date.now() / 200) * 5;
        ctx.fillStyle = `rgba(255, 255, 255, ${0.5 + Math.sin(Date.now()/200)*0.5})`;
        ctx.font = "bold 16px Segoe UI";
        ctx.fillText("CLICK ANYWHERE TO START", canvas.width/2, 280 + pulse);
    }

    function drawHearts() {
        ctx.fillStyle = "#ff4b2b";
        for(let i=0; i<lives; i++) {
            const hX = 30 + (i * 25);
            const hY = 25;
            ctx.beginPath();
            ctx.arc(hX-4, hY, 4, 0, Math.PI, true);
            ctx.arc(hX+4, hY, 4, 0, Math.PI, true);
            ctx.lineTo(hX, hY+8);
            ctx.fill();
        }
    }

    function update() {
        if(gameState !== "PLAYING") return;

        // Ball movement
        x += dx;
        y += dy;

        // Wall collisions
        if(x + dx > canvas.width-ballRadius || x + dx < ballRadius) {
            dx = -dx;
            playSound(300, 'sine');
        }
        if(y + dy < ballRadius) {
            dy = -dy;
            playSound(300, 'sine');
        } else if(y + dy > canvas.height-ballRadius) {
            if(x > paddleX && x < paddleX + paddleWidth) {
                dy = -dy;
                playSound(150, 'triangle');
            } else {
                lives--;
                playSound(100, 'sawtooth');
                if(lives <= 0) gameState = "GAMEOVER";
                else {
                    x = canvas.width/2; y = canvas.height-35;
                    dx = 4 + level; dy = -(4 + level);
                }
            }
        }

        // Paddle movement
        if(rightPressed && paddleX < canvas.width-paddleWidth) paddleX += 7;
        else if(leftPressed && paddleX > 0) paddleX -= 7;

        // Brick collision
        for(let c=0; c<brickColumnCount; c++) {
            for(let r=0; r<brickRowCount; r++) {
                let b = bricks[c][r];
                if(b.status == 1) {
                    if(x > b.x && x < b.x+brickWidth && y > b.y && y < b.y+brickHeight) {
                        dy = -dy;
                        b.status = 0;
                        score++;
                        playSound(500, 'sine');
                        if(score == brickRowCount * brickColumnCount) {
                            level++;
                            score = 0;
                            initBricks();
                            x = canvas.width/2; y = canvas.height-35;
                            dx = 4 + level; dy = -(4 + level);
                        }
                    }
                }
            }
        }
    }

    function draw() {
        if(gameState === "START") {
            drawStartScreen();
        } else if (gameState === "GAMEOVER") {
            ctx.fillStyle = "rgba(0,0,0,0.7)";
            ctx.fillRect(0,0,canvas.width, canvas.height);
            ctx.fillStyle = "red";
            ctx.font = "bold 40px Segoe UI";
            ctx.textAlign = "center";
            ctx.fillText("GAME OVER", canvas.width/2, 200);
            ctx.fillStyle = "white";
            ctx.font = "16px Segoe UI";
            ctx.fillText("Final Level: " + level, canvas.width/2, 240);
            ctx.fillText("CLICK TO TRY AGAIN", canvas.width/2, 300);
        } else {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Draw Bricks
            let color = levelColors[(level-1) % levelColors.length];
            for(let c=0; c<brickColumnCount; c++) {
                for(let r=0; r<brickRowCount; r++) {
                    if(bricks[c][r].status == 1) {
                        let bx = (c*(brickWidth+brickPadding))+brickOffsetLeft;
                        let by = (r*(brickHeight+brickPadding))+brickOffsetTop;
                        bricks[c][r].x = bx;
                        bricks[c][r].y = by;
                        ctx.fillStyle = color;
                        ctx.beginPath();
                        ctx.roundRect(bx, by, brickWidth, brickHeight, 4);
                        ctx.fill();
                        ctx.fillStyle = "rgba(255,255,255,0.1)";
                        ctx.fillRect(bx, by, brickWidth, 5);
                    }
                }
            }

            // Draw Ball
            ctx.beginPath();
            ctx.arc(x, y, ballRadius, 0, Math.PI*2);
            ctx.shadowBlur = 15;
            ctx.shadowColor = "#fff";
            ctx.fillStyle = "#fff";
            ctx.fill();
            ctx.shadowBlur = 0;

            // Draw Paddle
            ctx.fillStyle = "#00d2ff";
            ctx.beginPath();
            ctx.roundRect(paddleX, canvas.height-25, paddleWidth, paddleHeight, 6);
            ctx.fill();

            // UI
            drawHearts();
            ctx.fillStyle = "white";
            ctx.font = "bold 14px Segoe UI";
            ctx.textAlign = "right";
            ctx.fillText("LEVEL " + level, canvas.width - 25, 30);
            
            update();
        }
        requestAnimationFrame(draw);
    }

    draw();
</script>
</body>
</html>
