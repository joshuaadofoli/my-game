<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="theme-color" content="#000000">
<title>Ultimate Shooter Game</title>

<style>
body {
  margin: 0;
  background: linear-gradient(#111, #222);
  color: white;
  text-align: center;
  font-family: Arial;
  touch-action: manipulation;
}

#game {
  width: 400px;
  height: 500px;
  background: radial-gradient(circle, #000, #111);
  margin: auto;
  position: relative;
  overflow: hidden;
  border: 2px solid cyan;
  box-shadow: 0 0 15px cyan;
}

.player {
  width: 40px;
  height: 40px;
  background: cyan;
  position: absolute;
  bottom: 10px;
  left: 180px;
  border-radius: 8px;
}

.bullet {
  width: 6px;
  height: 12px;
  background: yellow;
  position: absolute;
}

.enemy { position: absolute; top: 0; }
.enemy.normal { width: 40px; height: 40px; background: red; }

#overlay {
  position: absolute;
  top: 40%;
  width: 100%;
  font-size: 24px;
  display: none;
}

button {
  margin: 5px;
  padding: 10px;
  font-size: 16px;
}
</style>
</head>

<body>

<h2>🔥 Ultimate Shooter</h2>

<p>
Score: <span id="score">0</span> |
High Score: <span id="highScore">0</span> |
Lives: <span id="lives">3</span>
</p>

<div id="game">
  <div class="player" id="player"></div>
  <div id="overlay"></div>
</div>

<div>
  <button onclick="togglePause()">⏸ Pause</button>
  <button onclick="restartGame()">🔄 Restart</button>
</div>

<div>
  <button onclick="moveLeft()">⬅️</button>
  <button onclick="shoot()">🔫</button>
  <button onclick="moveRight()">➡️</button>
</div>

<audio id="shootSound" src="https://www.soundjay.com/mechanical/sounds/mechanical-gunshot-01.mp3"></audio>
<audio id="gameOverSound" src="https://www.soundjay.com/misc/sounds/fail-buzzer-02.mp3"></audio>
<audio id="bgMusic" src="https://www.soundjay.com/free-music/sounds/space-ambient-1.mp3" loop></audio>

<script>
const game = document.getElementById("game");
const player = document.getElementById("player");
const overlay = document.getElementById("overlay");

const scoreText = document.getElementById("score");
const livesText = document.getElementById("lives");
const highScoreText = document.getElementById("highScore");

const shootSound = document.getElementById("shootSound");
const gameOverSound = document.getElementById("gameOverSound");
const bgMusic = document.getElementById("bgMusic");

let score, lives, playerX;
let gameRunning = true;
let paused = false;
let enemySpawner;

let highScore = localStorage.getItem("highScore") || 0;
highScoreText.textContent = highScore;

document.body.addEventListener("click", () => {
  bgMusic.play();
}, { once: true });

function startGame() {
  score = 0;
  lives = 3;
  playerX = 180;
  gameRunning = true;
  paused = false;

  scoreText.textContent = score;
  livesText.textContent = lives;

  overlay.style.display = "none";

  document.querySelectorAll(".enemy, .bullet").forEach(e => e.remove());

  spawnEnemies();
}

function togglePause() {
  paused = !paused;

  if (paused) {
    overlay.innerText = "⏸ PAUSED";
    overlay.style.display = "block";
    bgMusic.pause();
  } else {
    overlay.style.display = "none";
    bgMusic.play();
  }
}

document.addEventListener("keydown", e => {
  if (!gameRunning || paused) return;

  if (e.key === "ArrowLeft" && playerX > 0) playerX -= 20;
  if (e.key === "ArrowRight" && playerX < 360) playerX += 20;
  if (e.key === " ") shoot();

  player.style.left = playerX + "px";
});

function moveLeft() {
  if (paused) return;
  playerX -= 20;
  player.style.left = playerX + "px";
}

function moveRight() {
  if (paused) return;
  playerX += 20;
  player.style.left = playerX + "px";
}

function shoot() {
  if (paused) return;

  shootSound.currentTime = 0;
  shootSound.play();

  const bullet = document.createElement("div");
  bullet.className = "bullet";
  bullet.style.left = playerX + 18 + "px";
  bullet.style.bottom = "50px";
  game.appendChild(bullet);

  const move = setInterval(() => {
    if (paused) return;

    bullet.style.bottom = (parseInt(bullet.style.bottom) + 10) + "px";

    if (parseInt(bullet.style.bottom) > 500) {
      bullet.remove();
      clearInterval(move);
    }
  }, 50);
}

function spawnEnemies() {
  enemySpawner = setInterval(() => {
    if (!gameRunning || paused) return;

    const enemy = document.createElement("div");
    enemy.className = "enemy normal";
    enemy.style.left = Math.random() * 360 + "px";
    game.appendChild(enemy);

    const fall = setInterval(() => {
      if (paused) return;

      enemy.style.top = (enemy.offsetTop + 4) + "px";

      if (enemy.offsetTop > 460) {
        enemy.remove();
        loseLife();
        clearInterval(fall);
      }
    }, 50);

  }, 1000);
}

function loseLife() {
  lives--;
  livesText.textContent = lives;
  if (lives <= 0) gameOver();
}

function gameOver() {
  gameRunning = false;
  overlay.innerText = "💀 GAME OVER";
  overlay.style.display = "block";

  bgMusic.pause();
  gameOverSound.play();

  if (score > highScore) {
    localStorage.setItem("highScore", score);
    highScoreText.textContent = score;
  }

  clearInterval(enemySpawner);
}

function restartGame() {
  startGame();
}

startGame();
</script>

</body>
</html>
