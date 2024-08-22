<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Snake Game</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f4f4f4;
            font-family: Arial, sans-serif;
        }
        .container {
            text-align: center;
        }
        .menu, .game, .game-over {
            display: none;
        }
        .menu {
            display: block;
        }
        .menu-btn {
            padding: 10px 20px;
            font-size: 1.5em;
            cursor: pointer;
            margin-top: 20px;
        }
        canvas {
            border: 1px solid #000;
            background-color: #eee;
        }
        .game-over-buttons {
            margin-top: 20px;
        }
        .score {
            font-size: 1.2em;
            margin: 10px 0;
        }
        .scoreboard {
            margin-top: 20px;
            font-size: 1.1em;
        }
        .credits {
            font-size: 0.9em;
            color: #666;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Menú principal -->
        <div class="menu" id="menu">
            <h1>Snake Game</h1>
            <button class="menu-btn" id="play">Play</button>
            <div class="credits">Hecho por Valentín (Matrox017)</div>
        </div>

        <!-- Juego -->
        <div class="game" id="game">
            <canvas id="snakeCanvas" width="400" height="400"></canvas>
            <div class="score" id="score">Manzanas comidas: 0</div>
        </div>

        <!-- Fin del juego -->
        <div class="game-over" id="game-over">
            <div class="message" id="game-over-message">¡Juego terminado!</div>
            <div class="scoreboard">
                <h2>Tabla de Calificación</h2>
                <ul id="leaderboard">
                    <li>AlphaGamer - 63 puntos</li>
                    <li>BetaPlayer - 50 puntos</li>
                    <li>GammaPro - 45 puntos</li>
                    <li>DeltaMaster - 30 puntos</li>
                    <li>EpsilonAce - 20 puntos</li>
                </ul>
            </div>
            <div class="game-over-buttons">
                <button class="menu-btn" id="back-to-menu">Volver al Menú</button>
                <button class="menu-btn" id="restart">Reiniciar Juego</button>
            </div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('snakeCanvas');
        const ctx = canvas.getContext('2d');

        const gridSize = 20;
        const tileSize = canvas.width / gridSize;
        let snake = Array.from({ length: 4 }, (_, i) => ({ x: 10 - i, y: 10 }));
        let food = [];
        let dx = 1;
        let dy = 0;
        let isGameActive = false;
        let appleCount = 0; // Contador de manzanas comidas

        document.getElementById('play').addEventListener('click', startGame);
        document.getElementById('back-to-menu').addEventListener('click', goToMenu);
        document.getElementById('restart').addEventListener('click', restartGame);
        document.addEventListener('keydown', changeDirection);

        function startGame() {
            document.getElementById('menu').style.display = 'none';
            document.getElementById('game').style.display = 'block';
            document.getElementById('game-over').style.display = 'none';
            resetGame();
            isGameActive = true;
            gameLoop();
        }

        function goToMenu() {
            document.getElementById('menu').style.display = 'block';
            document.getElementById('game').style.display = 'none';
            document.getElementById('game-over').style.display = 'none';
        }

        function restartGame() {
            resetGame();
            document.getElementById('game-over').style.display = 'none';
            document.getElementById('game').style.display = 'block';
            isGameActive = true;
            gameLoop();
        }

        function gameLoop() {
            if (!isGameActive) return;
            moveSnake();
            if (checkCollision()) {
                isGameActive = false;
                document.getElementById('game').style.display = 'none';
                document.getElementById('game-over').style.display = 'block';
                return;
            }
            if (eatFood()) {
                growSnake();
                appleCount++;
                document.getElementById('score').textContent = `Manzanas comidas: ${appleCount}`;
                generateFood(); // Genera una nueva manzana
            }
            draw();
            setTimeout(gameLoop, 100);
        }

        function changeDirection(e) {
            switch (e.key) {
                case 'ArrowUp':
                    if (dy === 0) {
                        dx = 0;
                        dy = -1;
                    }
                    break;
                case 'ArrowDown':
                    if (dy === 0) {
                        dx = 0;
                        dy = 1;
                    }
                    break;
                case 'ArrowLeft':
                    if (dx === 0) {
                        dx = -1;
                        dy = 0;
                    }
                    break;
                case 'ArrowRight':
                    if (dx === 0) {
                        dx = 1;
                        dy = 0;
                    }
                    break;
            }
        }

        function moveSnake() {
            const head = { x: snake[0].x + dx, y: snake[0].y + dy };
            snake = [head, ...snake.slice(0, -1)];
        }

        function checkCollision() {
            const head = snake[0];
            return (
                head.x < 0 || head.x >= gridSize || head.y < 0 || head.y >= gridSize ||
                snake.slice(1).some(segment => segment.x === head.x && segment.y === head.y)
            );
        }

        function eatFood() {
            const head = snake[0];
            for (let i = 0; i < food.length; i++) {
                if (food[i].x === head.x && food[i].y === head.y) {
                    food.splice(i, 1); // Elimina la manzana comida
                    return true;
                }
            }
            return false;
        }

        function growSnake() {
            snake.push({...snake[snake.length - 1]});
        }

        function generateFood() {
            const freeSpaces = [];
            for (let x = 0; x < gridSize; x++) {
                for (let y = 0; y < gridSize; y++) {
                    if (!snake.some(segment => segment.x === x && segment.y === y) &&
                        !food.some(fruit => fruit.x === x && fruit.y === y)) {
                        freeSpaces.push({ x, y });
                    }
                }
            }
            if (freeSpaces.length === 0) return;
            const randomIndex = Math.floor(Math.random() * freeSpaces.length);
            food.push(freeSpaces[randomIndex]);
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawGrid();
            drawSnake();
            drawFood();
        }

        function drawGrid() {
            ctx.strokeStyle = '#ccc';
            ctx.lineWidth = 1;
            for (let i = 0; i < canvas.width; i += tileSize) {
                ctx.beginPath();
                ctx.moveTo(i, 0);
                ctx.lineTo(i, canvas.height);
                ctx.stroke();
            }
            for (let j = 0; j < canvas.height; j += tileSize) {
                ctx.beginPath();
                ctx.moveTo(0, j);
                ctx.lineTo(canvas.width, j);
                ctx.stroke();
            }
        }

        function drawSnake() {
            ctx.fillStyle = 'green';
            snake.forEach(segment => {
                ctx.fillRect(segment.x * tileSize, segment.y * tileSize, tileSize, tileSize);
            });
        }

        function drawFood() {
            ctx.fillStyle = 'red';
            food.forEach(fruit => {
                ctx.fillRect(fruit.x * tileSize, fruit.y * tileSize, tileSize, tileSize);
            });
        }

        function resetGame() {
            snake = Array.from({ length: 4 }, (_, i) => ({ x: 10 - i, y: 10 }));
            food = [];
            appleCount = 0; // Contador de manzanas comidas inicia en 0
            document.getElementById('score').textContent = `Manzanas comidas: ${appleCount}`;
            for (let i = 0; i < 3; i++) {
                generateFood(); // Genera 3 manzanas al inicio
            }
            dx = 1;
            dy = 0;
            isGameActive = true;
        }
    </script>
</body>
</html>
