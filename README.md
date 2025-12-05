<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Alien Dodge</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: radial-gradient(circle, #0b0b1e, #000);
      color: white;
      font-family: "Comic Sans MS", sans-serif;
    }
    #gameCanvas {
      display: block;
      margin: 0 auto;
      background: #111;
      border: 3px solid white;
      border-radius: 20px;
    }
    #hud { position:absolute; top:10px; left:10px; font-size:22px; }
    #startScreen, #shopScreen {
      position:absolute; inset:0;
      display:flex; flex-direction:column;
      justify-content:center; align-items:center;
      background:rgba(0,0,0,0.8);
      color:white; font-size:40px;
    }
    .btn {
      padding:15px 30px;
      font-size:24px;
      margin:10px;
      border:none;
      border-radius:15px;
      cursor:pointer;
      background:#4caf50;
      color:white;
    }
  </style>
</head>
<body>
  <div id="startScreen">
    <h1>👽 Alien Dodge!</h1>
    <button id="playBtn" class="btn">Play</button>
    <button id="shopBtn" class="btn">Shop</button>
  </div>

  <div id="shopScreen" style="display:none;">
    <h1>🛒 Alien Shop</h1>
    <button id="backBtn" class="btn">Back</button>
  </div>

  <canvas id="gameCanvas" width="800" height="600"></canvas>
  <div id="hud">Score: <span id="score">0</span></div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const startScreen = document.getElementById('startScreen');
const shopScreen = document.getElementById('shopScreen');
const playBtn = document.getElementById('playBtn');
const shopBtn = document.getElementById('shopBtn');
const backBtn = document.getElementById('backBtn');
\ nlet gameRunning = false;

// Player is now a cartoony alien
const player = { x:400, y:550, size:30, speed:5 };
let rocks = [], bullets = [], powerUps = [];
let score = 0, gameOver = false;
let baseRockSpeed = 2, currentRockSpeed = 2;
let powerUpActive = null, powerUpTimer = 0;
const keys = {};

window.addEventListener('keydown', e => keys[e.key] = true);
window.addEventListener('keyup', e => keys[e.key] = false);

function startGame(){
  startScreen.style.display="none";
  shopScreen.style.display="none";
  gameRunning = true;
  score = 0;
  rocks = []; bullets = []; powerUps = [];
  update();
}

function spawnRock(){
  rocks.push({ x:Math.random()*canvas.width, y:-20, size:20+Math.random()*20, speed:currentRockSpeed+Math.random()*2 });
}
function spawnPowerUp(){
  const types=['shoot','speed','slow'];
  powerUps.push({ x:Math.random()*760, y:Math.random()*400, type:types[Math.floor(Math.random()*3)] });
}

function update(){
  if(!gameRunning) return;

  // Movement
  if(keys['ArrowLeft']||keys['a']) player.x -= player.speed;
  if(keys['ArrowRight']||keys['d']) player.x += player.speed;
  if(keys['ArrowUp']||keys['w']) player.y -= player.speed;
  if(keys['ArrowDown']||keys['s']) player.y += player.speed;

  player.x = Math.max(0, Math.min(canvas.width-player.size, player.x));
  player.y = Math.max(0, Math.min(canvas.height-player.size, player.y));

  // Shooting
  if(powerUpActive === 'shoot' && keys[' ']){
    if(bullets.length < 5){ bullets.push({ x:player.x+player.size/2, y:player.y, speed:8 }); }
  }
  bullets.forEach(b => b.y -= b.speed);
  bullets = bullets.filter(b => b.y > 0);

  if(Math.random() < 0.03) spawnRock();
  rocks.forEach(r => r.y += r.speed);
  rocks = rocks.filter(r => r.y < canvas.height+20);

  if(Math.random() < 0.002 && powerUps.length < 3) spawnPowerUp();

  // Bullet collisions
  for(let i=0; i<rocks.length; i++){
    for(let j=0; j<bullets.length; j++){
      const dx=rocks[i].x-bullets[j].x, dy=rocks[i].y-bullets[j].y;
      if(Math.sqrt(dx*dx+dy*dy) < rocks[i].size/2){
        rocks.splice(i,1); bullets.splice(j,1); score+=5; break;
      }
    }
  }

  // Player hit
  for(let r of rocks){
    const dx=r.x-player.x, dy=r.y-player.y;
    if(Math.sqrt(dx*dx+dy*dy) < r.size/2 + player.size/2){ resetGame(); }
  }

  // Power-up pickup
  for(let i=0;i<powerUps.length;i++){
    const p=powerUps[i];
    if(Math.hypot(p.x-player.x, p.y-player.y) < 25){
      activatePowerUp(p.type); powerUps.splice(i,1); break;
    }
  }

  if(powerUpActive){
    powerUpTimer--;
    if(powerUpTimer<=0) deactivatePowerUp();
  }

  score++; document.getElementById('score').textContent = score;
  draw(); requestAnimationFrame(update);
}

function activatePowerUp(type){
  powerUpActive=type; powerUpTimer=60*5;
  if(type==='speed') player.speed *= 1.8;
  if(type==='slow') currentRockSpeed *= 0.5;
}
function deactivatePowerUp(){
  if(powerUpActive==='speed') player.speed /= 1.8;
  if(powerUpActive==='slow') currentRockSpeed = baseRockSpeed;
  powerUpActive=null;
}

function resetGame(){
  alert('You died! Final score: ' + score);
  gameRunning=false;
  startScreen.style.display="flex";
}

function draw(){
  ctx.clearRect(0,0,800,600);

  // Draw alien player
  ctx.fillStyle='lime';
  ctx.beginPath();
  ctx.ellipse(player.x+15, player.y+15, 20, 25, 0, 0, Math.PI*2);
  ctx.fill();

  // Eyes
  ctx.fillStyle='white';
  ctx.beginPath(); ctx.arc(player.x+10, player.y+10, 5, 0, Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.arc(player.x+20, player.y+10, 5, 0, Math.PI*2); ctx.fill();

  // Rocks
  ctx.fillStyle='gray';
  for(let r of rocks){ ctx.beginPath(); ctx.arc(r.x, r.y, r.size/2, 0, Math.PI*2); ctx.fill(); }

  // Bullets
  ctx.fillStyle='red';
  for(let b of bullets){ ctx.fillRect(b.x-2, b.y-10, 4, 10); }

  // Power-ups
  for(let p of powerUps){
    ctx.fillStyle = p.type==='shoot'?'red':p.type==='speed'?'yellow':'cyan';
    ctx.beginPath(); ctx.arc(p.x, p.y, 10, 0, Math.PI*2); ctx.fill();
  }
}

playBtn.onclick = startGame;
shopBtn.onclick = ()=>{ startScreen.style.display="none"; shopScreen.style.display="flex"; };
backBtn.onclick = ()=>{ shopScreen.style.display="none"; startScreen.style.display="flex"; };
</script>
</body>
</html>
