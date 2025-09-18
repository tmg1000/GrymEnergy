<!doctype html>
<html lang="sv">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Survival Runner - Browser Edition</title>
<style>
  html,body{height:100%;margin:0;background:linear-gradient(#162244,#55b9ff);font-family:Segoe UI,Roboto,Arial}
  #gameWrap{display:flex;align-items:center;justify-content:center;height:100%}
  canvas{background:linear-gradient(#162244,#55b9ff);box-shadow:0 8px 30px rgba(0,0,0,0.4);border-radius:8px}
  .ui{position:absolute;left:20px;top:20px;color:#fff}
  .menu{position:absolute;right:20px;top:20px;color:#fff;max-width:320px}
  .panel{background:rgba(20,20,28,0.85);padding:12px;border-radius:8px;margin-bottom:10px}
  button{padding:8px 12px;border-radius:8px;border:0;background:#2b7be9;color:#fff;cursor:pointer}
  input[type=text]{padding:6px;border-radius:6px;border:0}
  .hud{position:absolute;left:20px;bottom:20px;color:#fff}
  .small{font-size:13px;opacity:0.9}
  .leaderboard li{margin:4px 0}
</style>
</head>
<body>
<div id="gameWrap">
  <canvas id="c" width="1000" height="600"></canvas>
  <div class="menu" id="menu">
    <div class="panel" id="mainMenu">
      <h2>Survival Runner</h2>
      <div style="margin-bottom:8px">Name: <input id="nameInput" type="text" value="Player" /></div>
      <div style="display:flex;gap:8px">
        <button id="btnStart">Start Run</button>
        <button id="btnShop">Shop</button>
        <button id="btnBoard">Leaderboard</button>
      </div>
      <p class="small" style="margin-top:8px">Controls: A/D move, Space jump/hold for jetpack, E interact</p>
    </div>
    <div class="panel" id="shop" style="display:none">
      <h3>Shop</h3>
      <div>Coins: <span id="shopCoins">0</span></div>
      <div style="margin-top:8px"><button id="buyHat">Buy Color Hat (100)</button></div>
      <div style="margin-top:8px"><button id="backFromShop">Back</button></div>
    </div>
    <div class="panel" id="board" style="display:none">
      <h3>Top runs</h3>
      <ol class="leaderboard" id="leaderList"></ol>
      <div style="margin-top:8px"><button id="backFromBoard">Back</button></div>
    </div>
  </div>
  <div class="ui hud" id="hud" style="display:none">
    <div class="panel" style="width:420px">
      <div style="display:flex;justify-content:space-between"><div>Dist: <span id="dist">0</span>m</div><div>Coins: <span id="coins">0</span></div></div>
      <div style="display:flex;justify-content:space-between;margin-top:6px"><div>Health: <span id="hp">100</span></div><div>Jet: <span id="jet">0</span>s</div></div>
    </div>
  </div>
</div>
<script>
// -------------------------
// Constants & config
// -------------------------
const CANVAS_W = 1000, CANVAS_H = 600;
const GRAVITY = 2000; // px/s^2
const MOVE_SPEED = 300; // px/s
const JUMP_POWER = 740; // initial vel -px/s
const GROUND_HEIGHT = 90;
const PX_PER_METER = 100;
const MAX_LEADERBOARD = 20;
const ENEMY_BASE_SPEED = 140; // constant medium difficulty
const JETPACK_MAX = 5.0; // seconds of thrust
const JETPACK_DRAIN = 1.6; // multiplier
const MEDKIT_SPACING_METERS = 40;
const MAX_PIT_GAP = 140;

// -------------------------
// Persistence helpers (localStorage)
// -------------------------
const SAVE_KEY = 'survival_save_v1';
function defaultSave(){
  return {
    shop:{coins:0,owned:{hat_none:true},equipped:{hat:'hat_none',color:'blue'}},
    stats:{total_runs:0,total_coins:0,total_distance:0,longest_run:0},
    boards:{distance:[]}
  }
}
function loadSave(){
  try{ const raw = localStorage.getItem(SAVE_KEY); if(!raw){ const s=defaultSave(); saveSave(s); return s;} const s=JSON.parse(raw); if(!s.shop) s.shop=defaultSave().shop; if(!s.stats) s.stats=defaultSave().stats; if(!s.boards) s.boards=defaultSave().boards; s.boards.distance = s.boards.distance.slice(0,MAX_LEADERBOARD); return s }catch(e){ console.error('load fail',e); const s=defaultSave(); saveSave(s); return s }
}
function saveSave(s){ localStorage.setItem(SAVE_KEY, JSON.stringify(s)); }
function addDistanceLeader(name,dist){ const s=loadSave(); name=(name||'Player').trim()||'Player'; s.boards.distance.push({name,dist:Math.round(dist*100)/100,ts:Date.now()}); s.boards.distance.sort((a,b)=> (b.dist - a.dist) || (a.ts-b.ts)); s.boards.distance = s.boards.distance.slice(0,MAX_LEADERBOARD); saveSave(s); }

// -------------------------
// Game state
// -------------------------
const canvas = document.getElementById('c'); const ctx = canvas.getContext('2d');
let keys = {};
let lastTime = 0;
let running = false;
let camX = 0;

// world arrays
let groundSegments = []; // {x,w,y}
let platforms = [];
let medkits = [];
let coins = [];
let powerups = [];
let enemies = [];

// player
let player = null;

// HUD elements
const hud = document.getElementById('hud');
const distEl = document.getElementById('dist');
const coinsEl = document.getElementById('coins');
const hpEl = document.getElementById('hp');
const jetEl = document.getElementById('jet');

// menu controls
const menu = document.getElementById('menu');
const mainMenu = document.getElementById('mainMenu');
const shopPanel = document.getElementById('shop');
const boardPanel = document.getElementById('board');
const nameInput = document.getElementById('nameInput');
const btnStart = document.getElementById('btnStart');
const btnShop = document.getElementById('btnShop');
const btnBoard = document.getElementById('btnBoard');
const backFromShop = document.getElementById('backFromShop');
const backFromBoard = document.getElementById('backFromBoard');
const shopCoins = document.getElementById('shopCoins');
const buyHat = document.getElementById('buyHat');
const leaderList = document.getElementById('leaderList');

// setup events
window.addEventListener('keydown',e=>{ keys[e.key.toLowerCase()]=true; if(e.key===' '||e.code==='Space') e.preventDefault(); });
window.addEventListener('keyup',e=>{ keys[e.key.toLowerCase()]=false; });
canvas.addEventListener('click',()=>{ canvas.focus(); });

btnStart.addEventListener('click',()=>{ startRun(); });
btnShop.addEventListener('click',()=>{ mainMenu.style.display='none'; shopPanel.style.display='block'; refreshShop(); });
backFromShop.addEventListener('click',()=>{ shopPanel.style.display='none'; mainMenu.style.display='block'; });
btnBoard.addEventListener('click',()=>{ mainMenu.style.display='none'; boardPanel.style.display='block'; refreshBoard(); });
backFromBoard.addEventListener('click',()=>{ boardPanel.style.display='none'; mainMenu.style.display='block'; });
buyHat.addEventListener('click',()=>{ let s=loadSave(); if(s.shop.coins>=100 && !s.shop.owned['hat_color']){ s.shop.coins-=100; s.shop.owned['hat_color']=true; s.shop.equipped.color='green'; saveSave(s); refreshShop(); } else alert('Need 100 coins or already owned'); });

// -------------------------
// Helpers
// -------------------------
function randRange(a,b){ return a + Math.random()*(b-a); }
function rectsCollide(a,b){ return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.h + a.y > b.y; }

// -------------------------
// World generation
// -------------------------
function clearWorld(){ groundSegments = []; platforms=[]; medkits=[]; coins=[]; powerups=[]; enemies=[]; }
function extendGround(startX,endX,segW=96){ let x=startX; while(x<endX){ if(Math.random()<0.12 && x>startX+300){ let gapUnits = Math.floor(randRange(1,4)); let gap = segW*gapUnits; gap = Math.min(gap,MAX_PIT_GAP); x += gap; } else { groundSegments.push({x:Math.floor(x),w:segW,y:CANVAS_H-GROUND_HEIGHT}); x+=segW; } } }

function spawnMedkitsAndCoins(rangeStart,rangeEnd){ let minSpacing = MEDKIT_SPACING_METERS*PX_PER_METER; let x = rangeStart + 400; while(x < rangeEnd){ if(Math.random()<0.18){ let mx = x + randRange(0,minSpacing*1.25); mx = Math.floor(mx); let gy = CANVAS_H - GROUND_HEIGHT - 70 - randRange(0,120); medkits.push({x:mx,y:gy,used:false}); x = mx + minSpacing; } else { x += minSpacing*0.6; } }
 // coins
 for(let i=rangeStart;i<rangeEnd;i+=200){ if(Math.random()<0.5){ coins.push({x:i+100,y:CANVAS_H-GROUND_HEIGHT-50,collected:false}); } }
}

function spawnEnemies(rangeStart,rangeEnd){ for(let i=rangeStart+600;i<rangeEnd;i+=600){ if(Math.random()<0.6){ let ex=i + randRange(0,300); let gy = CANVAS_H-GROUND_HEIGHT-40; enemies.push({x:ex,y:gy,w:48,h:56,alive:true}); } } }

// -------------------------
// Player
// -------------------------
function createPlayer(shopState){ const startX = 120; const startY = CANVAS_H - GROUND_HEIGHT - 8; return {
  x:startX, y:startY, w:36, h:80, vx:0, vy:0, onGround:false, lastGroundTime:-999, lastJump:-999, health:100, jetFuel:0, jetEngaged:false, speedTimer:0, slowTimer:0, shop:shopState, startX:startX, maxDistance:0
}; }

function resetRun(){ clearWorld(); camX = 0; const worldWidth = 20000; extendGround(0, worldWidth); spawnMedkitsAndCoins(0, worldWidth); spawnEnemies(0, worldWidth); player = createPlayer(loadSave().shop); player.jetFuel = 0; player.health = 100; player.maxDistance = 0; }

// -------------------------
// Game loop
// -------------------------
function startRun(){ mainMenu.style.display='none'; hud.style.display='block'; running=true; resetRun(); lastTime = performance.now(); gameLoop(lastTime); }

function endRun(){ running=false; hud.style.display='none'; // submit score
 const meters = (player.x - player.startX)/PX_PER_METER; const name = nameInput.value || 'Player'; addDistanceLeader(name, meters); // update stats
 let s=loadSave(); s.stats.total_runs +=1; s.stats.total_distance += meters; s.stats.total_coins += coinsCollectedThisRun; s.stats.longest_run = Math.max(s.stats.longest_run, meters); s.shop.coins += coinsCollectedThisRun; saveSave(s); refreshShop(); alert('Run ended! Distance: ' + meters.toFixed(2) + ' m'); mainMenu.style.display='block'; }

let coinsCollectedThisRun = 0;

function gameLoop(now){ if(!running) return; const dt = Math.min(0.05, (now - lastTime)/1000); lastTime = now; update(dt); render(); requestAnimationFrame(gameLoop); }

function update(dt){ // player input & physics
 const left = keys['a']; const right = keys['d']; const jump = keys[' ']; const interact = keys['e'];
 // horizontal
 let move = 0; if(left) move -=1; if(right) move +=1; let speedMult = player.speedTimer>0?1.25:1.0; if(player.slowTimer>0) speedMult*=0.8; const targetV = MOVE_SPEED * speedMult * move; const accel=4800; if(player.vx < targetV) player.vx = Math.min(targetV, player.vx + accel*dt); else if(player.vx > targetV) player.vx = Math.max(targetV, player.vx - accel*dt);
 // gravity
 player.vy += GRAVITY*dt;
 // jump (coyote + buffer simpler: allow jump if on ground or shortly after)
 const canJump = player.onGround || (Date.now() - player.lastGroundTime) < 140;
 if(jump && canJump && (Date.now() - player.lastJump) > 150){ player.vy = -JUMP_POWER; player.onGround=false; player.lastJump = Date.now(); }
 // jetpack
 if(jump && player.jetFuel>0){ player.vy -= GRAVITY*0.58*dt; player.jetFuel = Math.max(0, player.jetFuel - dt*JETPACK_DRAIN); }
 // position
 player.x += player.vx * dt; player.y += player.vy * dt;
 // ground collision
 player.onGround = false;
 // find ground under player's feet
 const footX = player.x + player.w/2;
 let groundY = CANVAS_H - GROUND_HEIGHT;
 for(const g of groundSegments){ if(footX >= g.x && footX <= g.x + g.w){ groundY = g.y; break; } }
 if(player.y + player.h > groundY){ player.y = groundY - player.h; player.vy = 0; player.onGround = true; player.lastGroundTime = Date.now(); }

 // update camera
 camX = Math.max(0, player.x - 180);
 // distances
 const meters = Math.max(0,(player.x - player.startX)/PX_PER_METER);
 if(meters > player.maxDistance) player.maxDistance = meters;
 distEl.textContent = meters.toFixed(2);
 hpEl.textContent = Math.max(0, Math.round(player.health));
 jetEl.textContent = player.jetFuel.toFixed(2) + 's';

 // collect coins
 for(const c of coins){ if(!c.collected && Math.abs((c.x) - (player.x+player.w/2))<48 && Math.abs(c.y - (player.y+player.h/2))<60){ // press E to collect
   if(interact){ c.collected = true; coinsCollectedThisRun += 1; playSound('coin'); }
 } }
 coinsEl.textContent = (loadSave().shop.coins + coinsCollectedThisRun);

 // medkits - interact to heal
 for(const m of medkits){ if(!m.used && Math.abs(m.x - (player.x+player.w/2))<56 && Math.abs(m.y - (player.y+player.h/2))<80){ if(interact){ m.used = true; player.health = Math.min(100, player.health + 60); playSound('heal'); } } }

 // enemies movement and collisions
 for(const e of enemies){ if(!e.alive) continue; e.x -= ENEMY_BASE_SPEED * dt; // constant speed
 // collide with player
 if(Math.abs((e.x+e.w/2) - (player.x+player.w/2)) < (e.w/2 + player.w/2) && Math.abs((e.y+e.h/2) - (player.y+player.h/2)) < (e.h/2 + player.h/2)){
   // simple damage and bounce
   player.health -= 20; e.alive = false; playSound('hit'); if(player.health <= 0){ endRun(); return; }
 }
 }

 // offscreen or fallen
 if(player.y > CANVAS_H + 200){ endRun(); return; }
}

function playSound(name){ try{ const s = new Audio(name + '.wav'); s.play().catch(()=>{}); }catch(e){} }

// -------------------------
// Rendering
// -------------------------
function render(){ ctx.clearRect(0,0,CANVAS_W,CANVAS_H);
 // sky gradient
 const g=ctx.createLinearGradient(0,0,0,CANVAS_H); g.addColorStop(0,'#162244'); g.addColorStop(1,'#55b9ff'); ctx.fillStyle=g; ctx.fillRect(0,0,CANVAS_W,CANVAS_H);
 // parallax background
 for(let i=0;i<3;i++){ ctx.fillStyle = 'rgba(255,255,255,'+ (0.02*(i+1)) +')'; const rx = - (camX*0.2*(i+1)) % (CANVAS_W*2); ctx.fillRect(rx, 40 + i*40, CANVAS_W*2, 40); }
 // ground
 ctx.fillStyle='#5b3b25';
 for(const gseg of groundSegments){ const rX = Math.floor(gseg.x - camX); ctx.fillRect(rX, gseg.y, gseg.w, CANVAS_H - gseg.y); ctx.fillStyle='#6e492d'; ctx.fillRect(rX+6, gseg.y+6, gseg.w-12, CANVAS_H - (gseg.y+6)); ctx.fillStyle='#5b3b25'; }
 // platforms
 ctx.fillStyle='#ac7848'; for(const p of platforms){ const rX = Math.floor(p.x - camX); ctx.fillRect(rX, p.y, p.w, 16); }
 // coins
 for(const c of coins){ if(c.collected) continue; const rX = Math.floor(c.x - camX); ctx.beginPath(); ctx.arc(rX, c.y, 10, 0, Math.PI*2); ctx.fillStyle='#f0c828'; ctx.fill(); ctx.fillStyle='#fff3a0'; ctx.fill(); }
 // medkits
 for(const m of medkits){ if(m.used) continue; const rX = Math.floor(m.x - camX); ctx.fillStyle='#b22222'; ctx.fillRect(rX-20, m.y-14, 40, 28); ctx.fillStyle='#fff'; ctx.fillRect(rX-6, m.y-10, 12, 20); }
 // powerups
 for(const p of powerups){ if(p.used) continue; const rX=Math.floor(p.x-camX); ctx.fillStyle='#ffb347'; ctx.fillRect(rX-16,p.y-16,32,32); }
 // enemies
 ctx.fillStyle='#b33'; for(const e of enemies){ if(!e.alive) continue; const rX=Math.floor(e.x - camX); ctx.fillRect(rX, e.y - e.h, e.w, e.h); }
 // player
 ctx.fillStyle = '#2869e0'; if(loadSave().shop.equipped.color === 'green') ctx.fillStyle='#28b462'; else if(loadSave().shop.equipped.color === 'red') ctx.fillStyle='#c83b3b'; ctx.fillRect(Math.floor(player.x - camX), Math.floor(player.y), player.w, player.h);
 // simple hat
 ctx.fillStyle='#222'; ctx.fillRect(Math.floor(player.x - camX)+6, Math.floor(player.y)-10, player.w-12, 10);

 // HUD outlines
 ctx.fillStyle='rgba(0,0,0,0.2)'; ctx.fillRect(16,16,420,64);
}

// -------------------------
// Menu helpers
// -------------------------
function refreshShop(){ const s=loadSave(); shopCoins.textContent = s.shop.coins; }
function refreshBoard(){ const s=loadSave(); leaderList.innerHTML=''; for(const e of s.boards.distance){ const li = document.createElement('li'); li.textContent = e.name + ' — ' + e.dist + ' m'; leaderList.appendChild(li); } }

// initialize
(function init(){ canvas.width = CANVAS_W; canvas.height = CANVAS_H; refreshShop(); refreshBoard(); // focus canvas to get keys
 canvas.setAttribute('tabindex','0'); canvas.focus(); })();

</script>
</body>
</html>
