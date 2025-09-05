> [!chat] output a canvas.ephemeral-bloom html element unsepsecaped, then add \`\`\`js.run (include the .run this is special) self contained anonymous function that finds the first .ephemeral-bloom element to run snake inside it. add buttons for turning. use 🍎 as the emoji to use, ananymous js function... the \`\`\`js.run will autorun it
# Snake Game
<canvas class="ephemeral-bloom" width="400" height="400" style="border: 1px solid #ccc; background-color: #f0f0f0; display: block; margin-bottom: 10px;"></canvas>
<div class="snake-controls" style="display: flex; gap: 5px; justify-content: center; margin-bottom: 20px;">
    <button data-direction="up" style="padding: 10px 15px; font-size: 16px; cursor: pointer;">⬆️</button>
    <button data-direction="left" style="padding: 10px 15px; font-size: 16px; cursor: pointer;">⬅️</button>
    <button data-direction="down" style="padding: 10px 15px; font-size: 16px; cursor: pointer;">⬇️</button>
    <button data-direction="right" style="padding: 10px 15px; font-size: 16px; cursor: pointer;">➡️</button>
</div>

```js.run
(function() {
    const canvas = document.querySelector('.ephemeral-bloom');
    if (!canvas) {
        console.error('No .ephemeral-bloom element found to run snake in!');
        return;
    }

    const ctx = canvas.getContext('2d');
    const GRID_SIZE = 20; // Number of tiles across/down
    const TILE_SIZE = canvas.width / GRID_SIZE; // Size of each tile

    let snake = [];
    let food = {};
    let direction = 'right';
    let score = 0;
    let gameOver = false;
    let gameLoopInterval;
    const GAME_SPEED = 150; // milliseconds per frame (lower = faster)

    // Prepare font for drawing emojis
    ctx.font = `${TILE_SIZE * 0.9}px sans-serif`; // Adjust font size to fit tile
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';

    function initGame() {
        snake = [
            { x: 8, y: 10 },
            { x: 9, y: 10 },
            { x: 10, y: 10 }
        ];
        direction = 'right';
        score = 0;
        gameOver = false;
        generateFood();
        if (gameLoopInterval) {
            clearInterval(gameLoopInterval);
        }
        gameLoopInterval = setInterval(gameLoop, GAME_SPEED);
        draw();
    }

    function generateFood() {
        let newFoodPos;
        let collisionWithSnake;
        do {
            newFoodPos = {
                x: Math.floor(Math.random() * GRID_SIZE),
                y: Math.floor(Math.random() * GRID_SIZE)
            };
            // Ensure food doesn't spawn on the snake
            collisionWithSnake = snake.some(segment =>
                segment.x === newFoodPos.x && segment.y === newFoodPos.y
            );
        } while (collisionWithSnake);
        food = newFoodPos;
    }

    function draw() {
        // Clear canvas
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Draw score
        ctx.fillStyle = '#333';
        ctx.font = '20px sans-serif';
        ctx.fillText(`Score: ${score}`, canvas.width / 2, 25);
        ctx.font = `${TILE_SIZE * 0.9}px sans-serif`; // Reset font for game elements

        // Draw snake
        snake.forEach(segment => {
            ctx.fillText('🍎', segment.x * TILE_SIZE + TILE_SIZE / 2, segment.y * TILE_SIZE + TILE_SIZE / 2);
        });

        // Draw food
        ctx.fillText('🍎', food.x * TILE_SIZE + TILE_SIZE / 2, food.y * TILE_SIZE + TILE_SIZE / 2);

        // Draw Game Over message
        if (gameOver) {
            ctx.fillStyle = 'red';
            ctx.font = '40px sans-serif';
            ctx.fillText('Game Over!', canvas.width / 2, canvas.height / 2);
            ctx.font = '20px sans-serif';
            ctx.fillText('Click a direction button (or use arrow keys) to restart', canvas.width / 2, canvas.height / 2 + 40);
        }
    }

    function gameLoop() {
        if (gameOver) {
            clearInterval(gameLoopInterval);
            return;
        }

        const head = { x: snake[snake.length - 1].x, y: snake[snake.length - 1].y };

        // Update head position based on direction
        switch (direction) {
            case 'up': head.y--; break;
            case 'down': head.y++; break;
            case 'left': head.x--; break;
            case 'right': head.x++; break;
        }

        // Check for collisions
        const collidedWithWall = head.x < 0 || head.x >= GRID_SIZE || head.y < 0 || head.y >= GRID_SIZE;
        // Check for self-collision (excluding the very last segment which is the head itself)
        const collidedWithSelf = snake.some((segment, index) =>
            index < snake.length - 1 && segment.x === head.x && segment.y === head.y
        );

        if (collidedWithWall || collidedWithSelf) {
            gameOver = true;
            draw(); // Draw final state with "Game Over"
            clearInterval(gameLoopInterval);
            return;
        }

        snake.push(head); // Add new head

        // Check if food was eaten
        if (head.x === food.x && head.y === food.y) {
            score++;
            generateFood();
        } else {
            snake.shift(); // Remove tail if no food eaten
        }

        draw();
    }

    function changeDirection(newDir) {
        if (gameOver) {
            // If game is over, any valid direction button press restarts the game
            initGame();
            return;
        }

        const oppositeDirections = {
            'up': 'down',
            'down': 'up',
            'left': 'right',
            'right': 'left'
        };

        // Prevent immediate reversal unless the snake has only one segment (initial state)
        if (snake.length > 1 && newDir === oppositeDirections[direction]) {
            return;
        }
        direction = newDir;
    }

    // Event listeners for buttons
    const controlsDiv = document.querySelector('.snake-controls');
    if (controlsDiv) {
        controlsDiv.addEventListener('click', (event) => {
            const button = event.target.closest('button');
            if (button && button.dataset.direction) {
                changeDirection(button.dataset.direction);
            }
        });
    }

    // Optional: Keyboard controls for convenience
    document.addEventListener('keydown', (e) => {
        if (gameOver && ['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'w', 'a', 's', 'd'].includes(e.key.toLowerCase())) {
            initGame(); // Restart on arrow/WASD key press
            return;
        }
        switch (e.key.toLowerCase()) {
            case 'arrowup':
            case 'w':
                changeDirection('up');
                break;
            case 'arrowdown':
            case 's':
                changeDirection('down');
                break;
            case 'arrowleft':
            case 'a':
                changeDirection('left');
                break;
            case 'arrowright':
            case 'd':
                changeDirection('right');
                break;
        }
    });

    initGame(); // Start the game when the script runs
})();
```

> [!chat] 




