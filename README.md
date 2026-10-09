
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Racing Adventure 🏎️</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#10152d;color:white;font-family:Arial,sans-serif;text-align:center}
header{padding:15px 8px;background:linear-gradient(120deg,#712cf9,#ff3984)}
h1{margin:4px;font-size:29px}
header p{margin:5px;font-size:13px}
.game{max-width:440px;margin:auto;padding:12px}
.stats{display:flex;justify-content:space-around;gap:6px;margin-bottom:10px}
.stat{background:#242d4d;border-radius:12px;padding:10px 5px;flex:1;font-size:12px}
.stat b{display:block;font-size:20px;margin-top:5px;color:#ffe66d}
#road{height:min(58vh,490px);min-height:350px;position:relative;overflow:hidden;border:4px solid #58617c;border-radius:16px;background:#404653}
#road:before{content:"";position:absolute;left:10%;top:0;width:80%;height:100%;background:#343b49}
.lane{position:absolute;top:-100%;height:300%;width:5px;background:repeating-linear-gradient(to bottom,#fff 0 35px,transparent 35px 70px);animation:road 0.6s linear infinite}
.l1{left:36%}.l2{left:64%}
@keyframes road{to{transform:translateY(70px)}}
.car{position:absolute;width:45px;height:73px;border-radius:12px 12px 7px 7px;border:3px solid #f5f5f5;box-shadow:0 4px 7px #111}
.car:before{content:"";position:absolute;left:7px;right:7px;top:12px;height:20px;background:#a9e8ff;border-radius:5px 5px 2px 2px;border:2px solid #26364c}
.car:after{content:"";position:absolute;bottom:7px;left:5px;right:5px;height:5px;background:#ffeb86;border-radius:4px}
.player{bottom:16px;left:calc(50% - 22px);background:linear-gradient(90deg,#fb3484,#ff79ae);z-index:3}
.enemy{top:-85px;background:linear-gradient(90deg,#168cff,#65d5ff)}
.coin{position:absolute;width:28px;height:28px;background:#ffd43b;border:4px solid #fff1a1;border-radius:50%;box-shadow:0 0 10px #ffbf00;top:-40px;z-index:2}
.overlay{position:absolute;inset:0;z-index:5;background:#10152deF;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:22px}
.overlay h2{font-size:32px;color:#ff75bd;margin:8px}
.overlay p{line-height:1.5}
button{border:0;border-radius:30px;padding:13px 20px;font-size:16px;font-weight:bold;background:#ff3984;color:white;cursor:pointer;touch-action:manipulation}
.controls{display:flex;justify-content:space-between;gap:20px;margin:13px 0}
.controls button{flex:1;background:#293b69;font-size:25px;padding:13px}
#boost{background:#5b36df;font-size:14px}
.note{font-size:12px;color:#c6cde8}
</style>
</head>
<body>
<header>
<h1>🏎️ Racing Adventure</h1>
<p>Drive • Dodge • Collect • Win!</p>
</header>
<div class="game">
<div class="stats">
<div class="stat">⭐ SCORE<b id="score">0</b></div>
<div class="stat">🪙 COINS<b id="coins">0</b></div>
<div class="stat">🏆 BEST<b id="best">0</b></div>
</div>
<div id="road">
<div class="lane l1"></div><div class="lane l2"></div>
<div id="player" class="car player"></div>
<div id="overlay" class="overlay">
<h2>🏁 Ready to Race?</h2>
<p>Dodge the traffic, collect coins and beat your best score!</p>
<p>⬅️ ➡️ Move your car</p>
<button id="start">▶ START RACING</button>
</div>
</div>
<div class="controls">
<button id="left" aria-label="Move left">⬅️</button>
<button id="boost">⚡ SPEED BOOST</button>
<button id="right" aria-label="Move right">➡️</button>
</div>
<p class="note">Tip: Mobile par buttons dabao. Computer par arrow keys use karo!</p>
</div>
<script>
const road=document.getElementById("road");
const player=document.getElementById("player");
const overlay=document.getElementById("overlay");
const scoreEl=document.getElementById("score");
const coinsEl=document.getElementById("coins");
const bestEl=document.getElementById("best");
let score=0,coins=0,best=0,running=false,x=0;
let objects=[],last=0,spawn=0,coinSpawn=0,boostUntil=0,raf=0;
let keys={};
try{best=Number(localStorage.getItem("racingBest"))||0}catch(e){}
bestEl.textContent=best;
function reset(){
objects.forEach(o=>o.el.remove());objects=[];
score=0;coins=0;x=(road.clientWidth-45)/2;
scoreEl.textContent=0;coinsEl.textContent=0;
player.style.left=x+"px";spawn=0;coinSpawn=0;
boostUntil=0;last=0;
}
function start(){
cancelAnimationFrame(raf);reset();
overlay.style.display="none";running=true;
raf=requestAnimationFrame(loop);
}
function finish(){
running=false;
cancelAnimationFrame(raf);
if(score>best){best=score;bestEl.textContent=best;
try{localStorage.setItem("racingBest",best)}catch(e){}}
overlay.innerHTML='<h2>🏁 Race Over!</h2><p>⭐ Score: '+score+
'<br>🪙 Coins: '+coins+'<br>🏆 Best: '+best+
'</p><button id="again">↻ RACE AGAIN</button>';
overlay.style.display="flex";
document.getElementById("again").onclick=start;
}
function makeObject(type){
const el=document.createElement("div");
el.className=type==="coin"?"coin":"car enemy";
const max=road.clientWidth-(type==="coin"?28:45);
const pos=Math.random()*Math.max(0,max);
el.style.left=pos+"px";el.style.top="-80px";
road.appendChild(el);
objects.push({el,type,x:pos,y:-80});
}
function loop(now){
if(!running)return;
const dt=Math.min((now-last)/16.67||1,2);last=now;
const w=road.clientWidth;
if(keys.ArrowLeft||keys.a)x-=5*dt;
if(keys.ArrowRight||keys.d)x+=5*dt;
x=Math.max(3,Math.min(w-48,x));
player.style.left=x+"px";
const boosted=now<boostUntil;
const speed=(boosted?7:4)+Math.min(score/180,4);
spawn+=dt;coinSpawn+=dt;
if(spawn>Math.max(23,58-score/12)){makeObject("enemy");spawn=0}
if(coinSpawn>65){makeObject("coin");coinSpawn=0}
for(let i=objects.length-1;i>=0;i--){
const o=objects[i];o.y+=speed*dt;o.el.style.top=o.y+"px";
const pw=45,ph=73,ow=o.type==="coin"?28:45,oh=o.type==="coin"?28:73;
const hit=x<o.x+ow-5&&x+pw>o.x+5&&
o.y<road.clientHeight-16&&o.y+oh>road.clientHeight-16-ph;
if(hit){
if(o.type==="enemy"){finish();return}
coins++;score+=10;coinsEl.textContent=coins;
scoreEl.textContent=score;o.el.remove();objects.splice(i,1);continue;
}
if(o.y>road.clientHeight){o.el.remove();objects.splice(i,1);continue}
}
score+=0.08*dt;scoreEl.textContent=Math.floor(score);
raf=requestAnimationFrame(loop);
}
function move(dir){
if(!running)return;
x+=dir*28;x=Math.max(3,Math.min(road.clientWidth-48,x));
player.style.left=x+"px";
}
document.getElementById("start").onclick=start;
document.getElementById("left").onpointerdown=()=>move(-1);
document.getElementById("right").onpointerdown=()=>move(1);
document.getElementById("boost").onpointerdown=()=>{
if(running)boostUntil=performance.now()+1800;
};
document.addEventListener("keydown",e=>{
if(["ArrowLeft","ArrowRight"," "].includes(e.key))e.preventDefault();
keys[e.key]=true;
if(e.key===" "&&running)boostUntil=performance.now()+1800;
});
document.addEventListener("keyup",e=>keys[e.key]=false);
</script>
</body>
</html>
