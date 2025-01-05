//Impostazioni di base
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
canvas.width = 800;
canvas.height = 600;

// Caricamento delle risorse (immagini e suoni)
const resources = {
  playerImg: new Image(),
  enemyImg: new Image(),
  explosionImg: new Image(),
  explosionSound: new Audio('explosion-sound.mp3'),
  bgMusic: new Audio('background-music.mp3')
};

let gameOver = false;
let score = 0;
let level = 1;
let bgOffset = 0;
let enemies = [];
let explosions = [];
let player;
let enemySpeed = 2;
let enemyFrequency = 0.02;

// Funzione per gestire il caricamento delle risorse
function loadResources() {
  let loaded = 0;
  const totalResources = 4;

  resources.playerImg.src = 'https://example.com/your-airplane-image.png';
  resources.enemyImg.src = 'https://example.com/your-enemy-image.png';
  resources.explosionImg.src = 'https://example.com/explosion-image.png';
  resources.bgMusic.loop = true;

  // Aggiungi eventi di caricamento
  resources.playerImg.onload = resources.enemyImg.onload = resources.explosionImg.onload = () => {
    loaded++;
    if (loaded === totalResources) {
      resources.bgMusic.play();
      startGame();
    }
  };
}

// Inizializza il giocatore
function initPlayer() {
  player = {
    x: canvas.width / 2 - 16,
    y: canvas.height - 50,
    width: 32,
    height: 32,
    speed: 5,
    dx: 0
  };
}

// Gestione degli input dell'utente
function movePlayer(event) {
  if (gameOver) return;
  if (event.key === 'ArrowLeft') player.dx = -player.speed;
  if (event.key === 'ArrowRight') player.dx = player.speed;
}

function stopPlayer(event) {
  if (event.key === 'ArrowLeft' || event.key === 'ArrowRight') player.dx = 0;
}

// Funzione per disegnare il giocatore
function drawPlayer() {
  ctx.drawImage(resources.playerImg, player.x, player.y, player.width, player.height);
}

// Funzione per disegnare e muovere i nemici
function drawEnemies() {
  enemies.forEach(enemy => {
    enemy.move();
    ctx.drawImage(resources.enemyImg, enemy.x, enemy.y, enemy.width, enemy.height);
  });
}

// Generazione dei nemici
function generateEnemies() {
  if (Math.random() < enemyFrequency) {
    const enemyX = Math.random() * (canvas.width - 32);
    enemies.push(new Enemy(enemyX, 0, 'normal'));
  }
}

// Classe per nemici
class Enemy {
  constructor(x, y, type) {
    this.x = x;
    this.y = y;
    this.width = 32;
    this.height = 32;
    this.speed = enemySpeed;
    this.type = type;
  }

  move() {
    this.y += this.speed;
    if (this.type === 'fast') {
      this.x += Math.sin(this.y * 0.1) * 2;
    }
  }
}

// Gestione delle esplosioni
function createExplosion(x, y) {
  explosions.push({ x, y, width: 32, height: 32, frame: 0 });
}

function drawExplosions() {
  explosions.forEach((explosion, index) => {
    ctx.drawImage(resources.explosionImg, explosion.x, explosion.y, explosion.width, explosion.height);
    explosion.frame++;
    if (explosion.frame > 10) explosions.splice(index, 1);
  });
}

// Funzione di collisione
function checkCollisions() {
  enemies.forEach((enemy, index) => {
    if (player.x < enemy.x + enemy.width && player.x + player.width > enemy.x &&
        player.y < enemy.y + enemy.height && player.y + player.height > enemy.y) {
      createExplosion(enemy.x, enemy.y);
      enemies.splice(index, 1);
      gameOver = true;
      resources.explosionSound.play();
    }
  });
}

// Funzione per disegnare lo sfondo (con scorrimento)
function drawBackground() {
  bgOffset += 0.5;
  ctx.fillStyle = 'skyblue';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  ctx.fillStyle = 'rgba(255, 255, 255, 0.2)';
  ctx.fillRect(0, bgOffset % canvas.height, canvas.width, canvas.height);
  ctx.fillRect(0, (bgOffset - canvas.height) % canvas.height, canvas.width, canvas.height);
}

// Funzione per gestire la logica di gioco
function update() {
  if (gameOver) {
    ctx.fillStyle = 'white';
    ctx.font = '30px Arial';
    ctx.fillText(`Game Over! Score: ${score}. Press R to Restart.`, canvas.width / 2 - 150, canvas.height / 2);
    return;
  }

  // Aggiorna il movimento del giocatore
  player.x += player.dx;
  player.x = Math.max(0, Math.min(player.x, canvas.width - player.width)); // Prevent player from leaving the screen

  // Aumenta la difficoltà con il tempo
  if (score % 1000 === 0) {
    enemySpeed += 0.2;
    enemyFrequency += 0.005;
  }

  // Disegna gli elementi di gioco
  drawBackground();
  drawPlayer();
  generateEnemies();
  drawEnemies();
  drawExplosions();
  checkCollisions();

  // Aumenta il punteggio
  score++;

  requestAnimationFrame(update);
}

// Funzione per il riavvio del gioco
function restartGame() {
  if (gameOver && event.key === 'r') {
    gameOver = false;
    score = 0;
    level = 1;
    enemySpeed = 2;
    enemyFrequency = 0.02;
    bgOffset = 0;
    enemies = [];
    explosions = [];
    initPlayer();
    update();
}

// Inizializza gli event listener
document.addEventListener('keydown', movePlayer);
document.addEventListener('keyup', stopPlayer);
document.addEventListener('keydown', restartGame);

// Avvia il caricamento delle risorse
loadResources();