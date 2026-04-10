<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Game Vượt Chướng Ngại</title>
<style>
body{margin:0;overflow:hidden;background:black;font-family:sans-serif;}
#ui{position:absolute;top:10px;left:10px;color:white;}
#gameOver{
position:absolute;top:50%;left:50%;
transform:translate(-50%,-50%);
background:rgba(0,0,0,0.7);
color:white;padding:20px;border-radius:10px;
display:none;text-align:center;
}
button{padding:10px;margin-top:10px;}
#bxh{
position:absolute;right:10px;top:10px;
background:rgba(0,0,0,0.6);
color:white;padding:10px;border-radius:10px;
max-height:300px;overflow:auto;
}
</style>
</head>

<body>

<canvas id="game"></canvas>

<div id="ui">
<div id="distance">0 m</div>
<div id="level">Lv 1</div>
</div>

<div id="bxh">
<h3>🏆 BXH</h3>
<div id="bxhList"></div>
</div>

<div id="gameOver">
<h2>GAME OVER</h2>
<div id="finalScore"></div>
<input id="nameInput" placeholder="Tên bạn">
<br>
<button onclick="saveScore()">Lưu điểm</button>
<button onclick="restartGame()">Chơi lại</button>
</div>

<script>

// ===== CANVAS =====
let canvas = document.getElementById("game");
let ctx = canvas.getContext("2d");
canvas.width = innerWidth;
canvas.height = innerHeight;

// ===== ẢNH =====
// ===== LOAD ẢNH (FIX) =====
let imagesLoaded = 0;
let totalImages = 6;

function checkLoaded(){
  imagesLoaded++;
  if(imagesLoaded >= totalImages){
    console.log("Ảnh load xong");
  }
}

let sky = new Image();
sky.src = "img/sky.png";
sky.onload = checkLoaded;

let ground = new Image();
ground.src = "img/ground.png";
ground.onload = checkLoaded;

let playerImg = new Image();
playerImg.src = "img/player.png";
playerImg.onload = checkLoaded;

let heartImg = new Image();
heartImg.src = "img/heart.png";
heartImg.onload = checkLoaded;

let obsImgs = [
  "img/obs1.png",
  "img/obs2.png",
  "img/obs3.png"
].map(src=>{
  let i = new Image();
  i.src = src;
  i.onload = checkLoaded;
  return i;
});


// ===== GAME =====
let player = {x:100,y:300,vy:0};
let gravity = 0.6;
let jump = -15.5;

let obstacles = [];
let hearts = [];

let speed = 5;
let distance = 0;
let level = 1;

let lives = 3;
let maxLives = 5;

let gameRunning = true;

// nền
let groundX = 0;
let skyX = 0;

// ===== INPUT =====
addEventListener("touchstart", ()=> jumpPlayer());
addEventListener("keydown", e=>{
  if(e.code==="Space") jumpPlayer();
});

function jumpPlayer(){
  if(player.y >= canvas.height-150){
    player.vy = jump;
  }
}

// ===== SPAWN =====
setInterval(()=>{
  if(!gameRunning) return;

  let type = Math.floor(Math.random()*3);

  obstacles.push({
    x: canvas.width,
    y: canvas.height-150,
    w:80,
    h:80,
    img: obsImgs[type]
  });

  // tim hiếm
  if(Math.random()<0.2){
    hearts.push({
      x: canvas.width,
      y: canvas.height-200,
      w:40,h:40
    });
  }

},1500);

// ===== UPDATE =====
function update(){

  if(!gameRunning) return;

  // trọng lực
  player.vy += gravity;
  player.y += player.vy;

  if(player.y > canvas.height-150){
    player.y = canvas.height-150;
    player.vy = 0;
  }

  // obstacle
  obstacles.forEach(o=>{
    o.x -= speed;

    // va chạm
    if(player.x < o.x+o.w &&
       player.x+50 > o.x &&
       player.y < o.y+o.h &&
       player.y+50 > o.y){
        hit();
        o.x = -100;
    }
  });

  // heart
  hearts.forEach(h=>{
    h.x -= speed;

    if(player.x < h.x+h.w &&
       player.x+50 > h.x &&
       player.y < h.y+h.h &&
       player.y+50 > h.y){
        
        if(lives<maxLives) lives++;
        h.x=-100;
    }
  });

  // distance
  distance += speed;
  document.getElementById("distance").innerText = Math.floor(distance)+" m";

  // level
  if(distance>5000) level=2;
  if(distance>10000) level=3;
  if(distance>20000) level=4;
  if(distance>40000) level=5;

  document.getElementById("level").innerText = "Lv "+level;

  speed += 0.002;
}

// ===== HIT =====
let invincible = false;

function hit(){
  if(invincible) return;

  lives--;
  invincible = true;

  setTimeout(()=>invincible=false,1000);

  if(lives<=0){
    gameOver();
  }
}

// ===== DRAW =====
function draw(){
ctx.fillStyle = "white"; // hoặc màu trời
ctx.fillRect(0,0,canvas.width,canvas.height);

 // nền trời
if(sky.complete){
  ctx.drawImage(sky, skyX,0,canvas.width,canvas.height);
}

// đất
if(ground.complete){
  ctx.drawImage(ground, groundX,canvas.height-100,canvas.width,100);
  ctx.drawImage(ground, groundX+canvas.width,canvas.height-100,canvas.width,100);
}

// player
if(playerImg.complete){
  ctx.drawImage(playerImg, player.x, player.y,160,160);
}

  // hearts
  hearts.forEach(h=>{
    ctx.drawImage(heartImg,h.x,h.y,60,60);
    // obstacles (CHƯỚNG NGẠI VẬT)
  obstacles.forEach(o=>{
    if(o.img && o.img.complete){
      ctx.drawImage(o.img,o.x,o.y,o.w,o.h);
    } else {
      ctx.fillStyle="red";
      ctx.fillRect(o.x,o.y,o.w,o.h);
    }
  });
  });

  // lives
  for(let i=0;i<lives;i++){
    ctx.drawImage(heartImg,10+i*35,50,30,30);
  }
}

// ===== LOOP =====
function loop(){
  update();
  ctx.clearRect(0,0,canvas.width,canvas.height);
  draw();
  requestAnimationFrame(loop);
}
loop();

// ===== GAME OVER =====
function gameOver(){
  gameRunning = false;
  document.getElementById("gameOver").style.display="block";
  document.getElementById("finalScore").innerText = "Điểm: "+Math.floor(distance)+" m";
}

// ===== BXH =====
function getBXH(){
  return JSON.parse(localStorage.getItem("bxh")||"[]");
}

function saveBXH(data){
  localStorage.setItem("bxh",JSON.stringify(data));
}

function addScore(name,score){
  let bxh = getBXH();
  bxh.push({name,score});
  bxh.sort((a,b)=>b.score-a.score);
  bxh=bxh.slice(0,100);
  saveBXH(bxh);
}

function renderBXH(){
  let bxh=getBXH();
  let html="";
  bxh.forEach((p,i)=>{
    html+=`<div>#${i+1} ${p.name}: ${p.score}m</div>`;
  });
  document.getElementById("bxhList").innerHTML=html;
}

// ===== SAVE SCORE =====
function saveScore(){
  let name = document.getElementById("nameInput").value || "NoName";
  addScore(name, Math.floor(distance));
  renderBXH();
}

// ===== RESTART =====
function restartGame(){
  player.y=300;
  player.vy=0;

  obstacles=[];
  hearts=[];

  distance=0;
  speed=5;
  lives=3;
  level=1;

  gameRunning=true;

  document.getElementById("gameOver").style.display="none";
}

// load bxh
renderBXH();

</script>
</body>
</html>
