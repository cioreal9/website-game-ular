<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game Ular - Game Over</title>
    <style>
        body {
            background-color: #222;
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            flex-direction: column;
        }

        canvas {
            background-color: black;
            border: 2px solid white;
            margin-bottom: 20px;
        }

        .settings, #gameOverScreen {
            background-color: #333;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
            text-align: center;
            width: 300px;
            margin: 20px;
        }

        h2, h1 {
            color: #fff;
            margin-bottom: 10px;
        }

        .speed-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-bottom: 20px;
        }

        .speed-slider {
            width: 100%;
            margin: 10px 0;
        }

        .speed-value {
            color: #fff;
            font-size: 16px;
            font-weight: bold;
        }

        .start-button {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #ff7b00;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }

        .start-button:hover {
            background-color: #e56b00;
        }

        .controls, .controls2 {
            display: flex;
            justify-content: center;
            margin-top: 10px;
        }

        .button {
            width: 50px;
            height: 50px;
            margin: 5px;
            background-color: white;
            color: black;
            text-align: center;
            line-height: 50px;
            font-weight: bold;
            border: 2px solid black;
            cursor: pointer;
            user-select: none;
        }

        #scoreContainer {
            display: none;
            justify-content: center;
            align-items: center;
            background-color: #333;
            color: white;
            padding: 15px;
            border-radius: 10px;
            margin: 10px 0;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
            gap: 20px;
        }

        .scoreDisplay {
            font-size: 18px;
            font-weight: bold;
            text-align: center;
        }

        .scoreLabel {
            display: block;
            font-size: 14px;
            color: #bbb;
        }

        #gameOverScreen {
            display: none;
            background: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
        }

        #gameOverScreen button {
            padding: 10px 20px;
            font-size: 18px;
            background-color: lime;
            border: none;
            border-radius: 5px;
            color: black;
            font-weight: bold;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="settings" id="settings">
        <h2>Atur Kecepatan Game</h2>
        <div class="speed-container">
            <input type="range" id="speed" min="0" max="100" value="50" class="speed-slider">
            <span class="speed-value" id="speedValue">Kecepatan: 50</span>
        </div>
        <button id="startGame" class="start-button">Mulai Permainan</button>
    </div>
    <canvas id="snakeGame" width="400" height="400" style="display: none;"></canvas>
    <div id="controls" class="controls" style="display: none;">
        <div class="button" id="up">↑</div>
    </div>
    <div id="controls2" class="controls2" style="display: none;">
        <div class="button" id="left">←</div>
        <div class="button" id="down">↓</div>
        <div class="button" id="right">→</div>
    </div>
    <div id="scoreContainer">
        <div class="scoreDisplay">
            <span class="scoreLabel">Total Ekor</span>
            <span id="totalLength">0</span>
        </div>
        <div class="scoreDisplay">
            <span class="scoreLabel">Skor Tertinggi</span>
            <span id="highScoreDisplay">0</span>
        </div>
    </div>
    <div id="gameOverScreen">
        <h1>Game Over</h1>
        <p id="finalScore"></p>
        <button onclick="restartGame()">Main Lagi</button>
    </div>

    <script>
        const canvas = document.getElementById('snakeGame');
        const ctx = canvas.getContext('2d');
        const grid = 20;
        let snake = [{ x: 200, y: 200 }];
        let food = generateFood();
        let score = 0;
        let highScore = 0;
        let direction = { x: grid, y: 0 };
        let gameInterval;

        const highScoreDisplay = document.getElementById('highScoreDisplay');
        const totalLengthDisplay = document.getElementById('totalLength');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const finalScoreDisplay = document.getElementById('finalScore');
        const speedSlider = document.getElementById('speed');
        const speedValueDisplay = document.getElementById('speedValue');

        // Update speed value as slider changes
        speedSlider.oninput = () => {
            speedValueDisplay.textContent = 'Kecepatan: ' + speedSlider.value;
        };

        function getHighScore() {
            return localStorage.getItem('snakeHighScore') || 0;
        }

        function setHighScore(newHighScore) {
            localStorage.setItem('snakeHighScore', newHighScore);
        }

        window.onload = () => {
            highScore = getHighScore();
            highScoreDisplay.textContent = highScore;
        };

        document.getElementById('up').onclick = () => {
            if (direction.y === 0) direction = { x: 0, y: -grid };
        };
        document.getElementById('down').onclick = () => {
            if (direction.y === 0) direction = { x: 0, y: grid };
        };
        document.getElementById('left').onclick = () => {
            if (direction.x === 0) direction = { x: -grid, y: 0 };
        };
        document.getElementById('right').onclick = () => {
            if (direction.x === 0) direction = { x: grid, y: 0 };
        };

        document.getElementById('startGame').onclick = () => {
            const speed = parseInt(speedSlider.value);
            let gameSpeed;

            if (speed >= 0 && speed <= 30) {
                gameSpeed = 150;
            } else if (speed >= 31 && speed <= 60) {
                gameSpeed = 100;
            } else if (speed >= 61 && speed <= 100) {
                gameSpeed = 50;
            } else {
                alert('Kecepatan harus antara 0 dan 100.');
                return;
            }

            document.getElementById('settings').style.display = 'none';
            canvas.style.display = 'block';
            document.getElementById('controls').style.display = 'flex';
            document.getElementById('controls2').style.display = 'flex';
            document.getElementById('scoreContainer').style.display = 'flex';

            startGame(gameSpeed);
        };

        function startGame(speed) {
            if (gameInterval) {
                clearInterval(gameInterval);
            }
            snake = [{ x: 200, y: 200 }];
            direction = { x: grid, y: 0 };
            score = 0;
            food = generateFood();
            gameOverScreen.style.display = 'none';
            gameInterval = setInterval(gameLoop, speed);
        }

        function gameLoop() {
            const head = {
                x: (snake[0].x + direction.x + canvas.width) % canvas.width,
                y: (snake[0].y + direction.y + canvas.height) % canvas.height
            };

            if (isCollision(head)) {
                endGame();
                return;
            }

            snake.unshift(head);

            if (head.x === food.x && head.y === food.y) {
                score += food.type === 'red' ? 1 : 2;
                if (food.type === 'white') {
                    snake.push({ x: snake[snake.length - 1].x, y: snake[snake.length - 1].y });
                }
                food = generateFood();
            } else {
                snake.pop();
            }

            ctx.fillStyle = 'black';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = food.type === 'red' ? 'red' : 'white';
            ctx.beginPath();
            ctx.arc(food.x + grid / 2, food.y + grid / 2, grid / 2, 0, Math.PI * 2);
            ctx.fill();

            for (let i = 0; i < snake.length; i++) {
                const segment = snake[i];
                ctx.fillStyle = i === 0 ? 'yellow' : 'lime';
                ctx.beginPath();
                ctx.arc(segment.x + grid / 2, segment.y + grid / 2, grid / 2, 0, Math.PI * 2);
                ctx.fill();
            }

            totalLengthDisplay.textContent = snake.length - 1;

            if (score > highScore) {
                highScore = score;
                highScoreDisplay.textContent = highScore;
                setHighScore(highScore);
            }
        }

        function generateFood() {
            return {
                x: grid * Math.floor(Math.random() * 20),
                y: grid * Math.floor(Math.random() * 20),
                type: Math.random() < 0.5 ? 'red' : 'white'
            };
        }

        function isCollision(head) {
            for (let i = 1; i < snake.length; i++) {
                if (snake[i].x === head.x && snake[i].y === head.y) {
                    return true;
                }
            }
            return false;
        }

        function endGame() {
            clearInterval(gameInterval);
            finalScoreDisplay.textContent = 'Skor Akhir: ' + score;
            gameOverScreen.style.display = 'block';
        }

        function restartGame() {
            document.getElementById('settings').style.display = 'block';
            canvas.style.display = 'none';
            document.getElementById('controls').style.display = 'none';
            document.getElementById('controls2').style.display = 'none';
            document.getElementById('scoreContainer').style.display = 'none';
            gameOverScreen.style.display = 'none';
        }
    </script>
</body>
</html>
