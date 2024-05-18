# Snake-Game
# JavaScript Snake Game

This project is a simple implementation of the classic Snake game using HTML, CSS, and JavaScript. The game allows players to control a snake, eat apples to grow in size, and avoid colliding with walls or the snake's own body. 

## Table of Contents

- [Demo](#demo)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Code Overview](#code-overview)
  - [index.html](#indexhtml)
  - [style.css](#stylecss)
  - [script.js](#scriptjs)
- [Contributing](#contributing)
- [License](#license)

## Demo

Check out the live demo of the game [here](#).

## Features

- Dynamic game board using HTML5 Canvas.
- Responsive design with CSS.
- Basic game mechanics including snake movement, apple consumption, and collision detection.
- Score display and game over screen.

## Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/your-username/snake-game.git
    ```

2. Navigate to the project directory:
    ```bash
    cd snake-game
    ```

## Usage

To play the game, open the `index.html` file in your web browser:
```bash
open index.html


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="script.js"></script>
    <link rel="stylesheet" href="style.css">
    <script src="https://kit.fontawesome.com/163dbef831.js" crossorigin="anonymous"></script>
    <title>JavaScript Snake</title>
</head>
<body>
    <header>
        <h1>Snake</h1>
    </header>
</body>
</html>


body {
    margin: 0;
    font-family: Arial, sans-serif;
    background-color: #f0f0f0;
}

header {
    text-align: center;
    margin: 20px 0;
}

canvas {
    display: block;
    margin: 0 auto;
    border: 30px solid #B38B6D;
    background-color: #C8AD7F;
}


window.onload = () => {
    let canvasWidth = 900;
    let canvasHeight = 600;
    let blockSize = 30;
    let ctx;
    let delay = 100;
    let snakee;
    let applee;
    let widthInBlocks = canvasWidth / blockSize;
    let heightInBlocks = canvasHeight / blockSize;
    let score;
    let timeout;

    init();

    function init() {
        let canvas = document.createElement('canvas');
        canvas.width = canvasWidth;
        canvas.height = canvasHeight;
        canvas.style.border = "30px solid #B38B6D";
        canvas.style.display = "block";
        canvas.style.margin = "0 auto";
        canvas.style.backgroundColor = "#C8AD7F";
        document.body.appendChild(canvas);
        ctx = canvas.getContext('2d');
        snakee = new Snake([[6, 4], [5, 4], [4, 4]], "right");
        applee = new Apple([10, 10]);
        score = 0;
        refreshCanvas();
    }

    function refreshCanvas() {
        snakee.advance();
        if (snakee.checkCollision()) {
            gameOver();
        } else {
            if (snakee.isEatingApple(applee)) {
                score++;
                snakee.ateApple = true;
                do {
                    applee.setNewPosition();
                } while (applee.isOnSnake(snakee));
            }
            ctx.clearRect(0, 0, canvasWidth, canvasHeight);
            drawScore();
            snakee.draw();
            applee.draw();
            timeout = setTimeout(refreshCanvas, delay);
        }
    }

    function gameOver() {
        let centreX = canvasWidth / 2;
        let centreY = canvasHeight / 2;
        ctx.save();
        ctx.font = "bold 70px Helvetica";
        ctx.fillStyle = "rgb(255, 75, 75)";
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.strokeStyle = "#000";
        ctx.lineWidth = 5;
        ctx.strokeText("GAME OVER", centreX, centreY - 180);
        ctx.fillText("GAME OVER", centreX, centreY - 180);
        ctx.font = "bold 25px Helvetica";
        ctx.fillText("Press SPACE to restart", centreX, centreY - 120);
        ctx.restore();
    }

    function restart() {
        snakee = new Snake([[6, 4], [5, 4], [4, 4]], "right");
        applee = new Apple([10, 10]);
        score = 0;
        clearTimeout(timeout);
        refreshCanvas();
    }

    function drawScore() {
        let centreX = canvasWidth / 2;
        let centreY = canvasHeight / 2;
        ctx.save();
        ctx.font = "bold 250px sans-serif";
        ctx.fillStyle = "#B4A696";
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.fillText(score, centreX, centreY);
        ctx.restore();
    }

    function drawBlock(ctx, position) {
        let x = position[0] * blockSize;
        let y = position[1] * blockSize;
        ctx.fillRect(x, y, blockSize, blockSize);
    }

    function Snake(body, direction) {
        this.body = body;
        this.direction = direction;
        this.ateApple = false;

        this.draw = function () {
            ctx.save();
            ctx.fillStyle = "#B27B6D";
            for (let i = 0; i < this.body.length; i++) {
                drawBlock(ctx, this.body[i]);
            }
            ctx.restore();
        };

        this.advance = function () {
            let nextPosition = this.body[0].slice();
            switch (this.direction) {
                case "left":
                    nextPosition[0] -= 1;
                    break;
                case "right":
                    nextPosition[0] += 1;
                    break;
                case "up":
                    nextPosition[1] -= 1;
                    break;
                case "down":
                    nextPosition[1] += 1;
                    break;
                default:
                    throw ("Invalid direction");
            }
            this.body.unshift(nextPosition);
            if (!this.ateApple) {
                this.body.pop();
            } else {
                this.ateApple = false;
            }
        };

        this.setDirection = function (newDirection) {
            let allowedDirections;
            switch (this.direction) {
                case "left":
                case "right":
                    allowedDirections = ["up", "down"];
                    break;
                case "up":
                case "down":
                    allowedDirections = ["left", "right"];
                    break;
                default:
                    throw ("Invalid direction");
            }
            if (allowedDirections.indexOf(newDirection) > -1) {
                this.direction = newDirection;
            }
        };

        this.checkCollision = function () {
            let wallCollision = false;
            let snakeCollision = false;
            let head = this.body[0];
            let rest = this.body.slice(1);
            let snakeX = head[0];
            let snakeY = head[1];
            let minX = 0;
            let minY = 0;
            let maxX = widthInBlocks - 1;
            let maxY = heightInBlocks - 1;
            let isNotBetweenHorizontalWalls = snakeX < minX || snakeX > maxX;
            let isNotBetweenVerticalWalls = snakeY < minY || snakeY > maxY;
            if (isNotBetweenHorizontalWalls || isNotBetweenVerticalWalls) {
                wallCollision = true;
            }
            for (let i = 0; i < rest.length; i++) {
                if (snakeX === rest[i][0] && snakeY === rest[i][1]) {
                    snakeCollision = true;
                }
            }
            return wallCollision || snakeCollision;
        };

        this.isEatingApple = function (appleToEat) {
            let head = this.body[0];
            return head[0] === appleToEat.position[0] && head[1] === appleToEat.position[1];
        };
    }

    function Apple(position) {
        this.position = position;

        this.draw = function () {
            ctx.save();
            ctx.fillStyle = "#B38B6D";
            ctx.beginPath();
            let radius = blockSize / 2;
            let x = this.position[0] * blockSize + radius;
            let y = this.position[1] * blockSize + radius;
            ctx.arc(x, y, radius, 0, Math.PI * 2, true);
            ctx.fill();
            ctx.restore();
        };

        this.setNewPosition = function () {
            let newX = Math.round
