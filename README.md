<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Space Game</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: black;
            overflow: hidden;
        }
        
        #gamestartscreen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background-color: black;
            z-index: 10;
        }

        #splashimage {
            width: 100vw;
            height: 100vh;
            object-fit: cover;
        }

        #startButton {
            position: absolute;
            bottom: 10%;
            padding: 20px 40px;
            font-size: 28px;
            background-color: #ffcc00;
            border: none;
            cursor: pointer;
            border-radius: 10px;
            color: black;
            font-weight: bold;
        }

        #gamecontainer {
            display: none;
            position: absolute;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            text-align: center;
        }

        #score {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 20px;
            color: white;
        }

        #lives {
            position: absolute;
            top: 10px;
            right: 10px;
            font-size: 20px;
            color: white;
        }

        #gameover {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 48px;
            color: red;
        }

        #winScreen {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 48px;
            color: green;
            text-align: center;
            background: white;
            padding: 20px;
            border: 5px solid green;
            border-radius: 10px;
        }
    </style>
</head>
<body>

<div id="wrapper">
    <div id="gamestartscreen">
        <img src="MY_GAME/Saved_Pictures/SPLASH_SCREEN.jpg" alt="Splash Screen" id="splashimage">
        <button id="startButton">Start Game</button>
    </div>
    
    <div id="gamecontainer">
        <canvas id="gamecanvas"></canvas>
        <span id="score">Score: 0/10</span>
        <span id="lives">Lives: 3/3</span>
    </div>

    <div id="gameover">Game Over</div>

    <div id="winScreen">
        Success,portal opened and stable
        <br><br>
        <button onclick="restartGame()" style="font-size: 20px; padding: 10px 20px; background-color: yellow; border: none; cursor: pointer;">
            Play Again
        </button>
    </div>
</div>

<script>
    const canvas = document.getElementById('gamecanvas');
    const ctx = canvas.getContext('2d');
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    window.addEventListener('resize', function() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    });

    document.getElementById('startButton').addEventListener('click', function() {
        document.getElementById('gamestartscreen').style.display = 'none';
        document.getElementById('gamecontainer').style.display = 'block';
        requestAnimationFrame(gameLoop);
    });

    let score = 0;
    const totalCollectibles = 10;
    let lives = 3;
    const totalLives = 3;

    let character = { x: 300, y: canvas.height - 80, width: 40, height: 60, color: 'white', speed: 5, dy: 0, gravity: 0.6, grounded: true };
    let keys = {};
    let collectibles = [
        { x: 150, y: canvas.height - 40, width: 20, height: 20, color: 'yellow' },
        { x: 500, y: canvas.height - 40, width: 20, height: 20, color: 'yellow' },
        { x: 900, y: canvas.height - 40, width: 20, height: 20, color: 'yellow' },
        { x: 100, y: 300, width: 20, height: 20, color: 'yellow' },
        { x: 400, y: 200, width: 20, height: 20, color: 'yellow' },
        { x: 600, y: 400, width: 20, height: 20, color: 'yellow' },
        { x: 700, y: 150, width: 20, height: 20, color: 'yellow' },
        { x: 300, y: canvas.height - 80, width: 20, height: 20, color: 'yellow' },
        { x: 500, y: canvas.height - 120, width: 20, height: 20, color: 'yellow' },
		{ x: 150, y: canvas.height - 70, width: 20, height: 20, color: 'yellow' },
		{ x: 150, y: canvas.height - 100, width: 20, height: 20, color: 'yellow' },
        { x: 100, y: canvas.height - 160, width: 20, height: 20, color: 'yellow' }
    ];

    document.addEventListener('keydown', (e) => keys[e.key] = true);
    document.addEventListener('keyup', (e) => keys[e.key] = false);

    function moveCharacter() {
        if (keys['ArrowLeft']) character.x -= character.speed;
        if (keys['ArrowRight']) character.x += character.speed;
        if ((keys[' '] || keys['ArrowUp']) && character.grounded) jump();
        character.dy += character.gravity;
        character.y += character.dy;
        if (character.x < 0) character.x = 0;
        if (character.x + character.width > canvas.width) character.x = canvas.width - character.width; 
        character.grounded = false;
        checkPlatformCollision();
    }

    function jump() {
        character.dy = -15;
        character.grounded = false;
    }

    function drawCharacter() {
        ctx.fillStyle = character.color;
        ctx.fillRect(character.x, character.y, character.width, character.height);
    }

    function drawCollectibles() {
        collectibles.forEach(item => {
            ctx.fillStyle = item.color;
            ctx.fillRect(item.x, item.y, item.width, item.height);
        });
    }

    function checkCollectibles() {
        collectibles.forEach((item, index) => {
            if (character.x < item.x + item.width && character.x + character.width > item.x &&
                character.y < item.y + item.height && character.y + character.height > item.y) {
                collectibles.splice(index, 1);
                score++;
                document.getElementById('score').innerText = `Score: ${score}/${totalCollectibles}`;
            }
        });
    }

    function updateLives() {
        document.getElementById('lives').innerText = `Lives: ${lives}/${totalLives}`;
    }

    let platforms = [
        { x: 0, y: canvas.height - 20, width: canvas.width, height: 20 }, // Ground platform
        { x: 250, y: canvas.height - 100, width: 150, height: 20 },
        { x: 550, y: canvas.height - 180, width: 150, height: 20 },
        { x: 350, y: canvas.height - 260, width: 150, height: 20 },
        { x: 750, y: canvas.height - 340, width: 150, height: 20 },
        { x: 450, y: canvas.height - 420, width: 150, height: 20 },
        { x: 950, y: canvas.height - 500, width: 150, height: 20 },
        { x: 650, y: canvas.height - 580, width: 150, height: 20 },
        { x: 200, y: canvas.height - 660, width: 150, height: 20 },
        { x: 850, y: canvas.height - 740, width: 150, height: 20 }
    ];

    function drawPlatforms() {
        ctx.fillStyle = 'green';
        platforms.forEach(platform => ctx.fillRect(platform.x, platform.y, platform.width, platform.height));
    }

    function checkPlatformCollision() {
        let onGround = false;
        platforms.forEach(platform => {
            let charBottom = character.y + character.height;
            let charTop = character.y;
            let charLeft = character.x;
            let charRight = character.x + character.width;
            let platTop = platform.y;
            let platBottom = platform.y + platform.height;
            let platLeft = platform.x;
            let platRight = platform.x + platform.width;

            if (charBottom > platTop && charTop < platTop && charRight > platLeft && charLeft < platRight) {
                character.y = platTop - character.height;
                character.dy = 0;
                onGround = true;
            } else if (charBottom > platTop && charTop < platBottom && charRight > platLeft && charLeft < platRight) {
                character.y = platBottom;
                character.dy = Math.abs(character.dy);
            }
        });
        character.grounded = onGround;
    }

    let enemies = [
        { x: 200, y: 350, width: 30, height: 30, speed: 2, direction: 1 },
        { x: 600, y: 250, width: 30, height: 30, speed: 3, direction: -1 },
        { x: 400, y: 150, width: 30, height: 30, speed: 2.5, direction: 1 }
    ];

    function drawEnemies() {
        ctx.fillStyle = 'red';
        enemies.forEach(enemy => {
            ctx.beginPath();
            ctx.moveTo(enemy.x, enemy.y);
            ctx.lineTo(enemy.x + enemy.width / 2, enemy.y - enemy.height);
            ctx.lineTo(enemy.x + enemy.width, enemy.y);
            ctx.closePath();
            ctx.fill();
        });
    }

    function moveEnemies() {
        enemies.forEach(enemy => {
            enemy.x += enemy.speed * enemy.direction;
            if (enemy.x < 0 || enemy.x + enemy.width > canvas.width) {
                enemy.direction *= -1;
            }
        });
    }

    function checkEnemyCollision() {
        enemies.forEach(enemy => {
            if (
                character.x < enemy.x + enemy.width &&
                character.x + character.width > enemy.x &&
                character.y < enemy.y + enemy.height &&
                character.y + character.height > enemy.y
            ) {
                gameOver();
            }
        });
    }

    function gameOver() {
        lives--;
        updateLives();
        
        if (lives > 0) {
            alert(`You lost a life! Lives remaining: ${lives}`);
            resetPlayer();
            requestAnimationFrame(gameLoop);
        } else {
            endGame();
        }
    }

    function resetPlayer() {
        character.x = 300;
        character.y = 400;
        character.dy = 0;
    }

    function endGame() {
        document.getElementById('gamecontainer').style.display = 'none';
        document.getElementById('gameover').style.display = 'block';
    }

    function showWinScreen() {
        cancelAnimationFrame(gameLoopId);
        document.getElementById('gamecontainer').style.display = 'none';
        document.getElementById('winScreen').style.display = 'block';
    }

    function restartGame() {
        score = 0;
        lives = totalLives;
        character.x = 300;
        character.y = 400;
        character.dy = 0;
        updateLives();
        document.getElementById('score').innerText = `Score: ${score}/${totalCollectibles}`;
        
        document.getElementById('winScreen').style.display = 'none';
        document.getElementById('gameover').style.display = 'none';
        document.getElementById('gamecontainer').style.display = 'block';
        
        gameLoopId = requestAnimationFrame(gameLoop);
    }

    function gameLoop() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        moveCharacter();
        moveEnemies();
        drawPlatforms();
        drawCollectibles();
        checkCollectibles();
        drawEnemies();
        checkEnemyCollision();
        drawCharacter();

        if (score >= totalCollectibles) {
            showWinScreen();
        } else if (character.y > canvas.height) {
            gameOver();
        } else {
            gameLoopId = requestAnimationFrame(gameLoop);
        }
    }
</script>

</body>
</html>
