# doodle-world
from pathlib import Path
import zipfile, textwrap, os

out = Path("/mnt/data/doodle-world")
out.mkdir(exist_ok=True)

html = r'''<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Doodle World — click, drag & play</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Patrick+Hand:wght@400&family=Kalam:wght@400;700&display=swap');
:root{--paper:#fff9ed;--ink:#24231f;--yellow:#ffd15b;--pink:#ffb9c7;--blue:#b9e8f5;--green:#b8e5a0;--purple:#d9c7ff}
*{box-sizing:border-box}
body{margin:0;background:var(--paper);color:var(--ink);font-family:'Patrick Hand','Comic Sans MS',cursive;overflow-x:hidden}
header{position:sticky;top:0;z-index:10;background:rgba(255,249,237,.92);backdrop-filter:blur(10px);border-bottom:2px solid #28251f22;display:flex;align-items:center;justify-content:space-between;padding:13px clamp(16px,4vw,52px);gap:16px}
.brand{font-size:clamp(22px,3vw,32px);font-weight:bold;display:flex;align-items:center;gap:10px;white-space:nowrap}
.brand .star{font-size:30px;color:#f7bb20;text-shadow:1px 2px #222}
nav{display:flex;gap:8px;align-items:center}
nav button,.toolbtn{font:inherit;font-size:18px;border:0;background:transparent;padding:8px 13px;border-radius:999px;cursor:pointer;color:inherit}
nav button:hover,nav button.active{background:#ffdc7c}
.header-right{font-size:17px;white-space:nowrap}
main{max-width:1450px;margin:auto;padding:26px clamp(12px,3vw,40px) 55px}
.hero{text-align:center;position:relative;padding:28px 12px 18px;min-height:260px}
.kicker{font-size:18px;letter-spacing:1px;color:#686052}
h1{font-size:clamp(42px,6.5vw,76px);line-height:1.02;margin:10px 0 14px;font-weight:400}
.hero p{font-size:clamp(18px,2.2vw,23px);max-width:580px;margin:0 auto 23px;line-height:1.35}
.primary{font:inherit;font-size:21px;border:2px solid var(--ink);background:var(--yellow);padding:12px 24px;border-radius:999px;box-shadow:3px 4px 0 var(--ink);cursor:pointer;transition:transform .15s,box-shadow .15s}
.primary:hover{transform:translate(2px,2px);box-shadow:1px 2px 0 var(--ink)}
.doodle-field{position:relative;min-height:540px;border:2px dashed #cfc4ad;border-radius:28px;overflow:hidden;background:radial-gradient(#e8ddc9 1px,transparent 1px);background-size:19px 19px}
.hint{position:absolute;top:14px;left:18px;font-size:17px;color:#7c705e;z-index:2}
.doodle{position:absolute;display:grid;place-items:center;cursor:grab;user-select:none;touch-action:none;transition:filter .15s,transform .15s;filter:drop-shadow(1px 2px 0 #26231c22)}
.doodle:active{cursor:grabbing}
.doodle:hover{filter:drop-shadow(2px 4px 0 #26231c28)}
.doodle svg{width:100%;height:100%;overflow:visible}
.doodle .label{position:absolute;top:100%;font-size:16px;white-space:nowrap;transform:rotate(-5deg);pointer-events:none}
#sun{width:105px;height:105px;left:7%;top:10%}
#cloud{width:135px;height:90px;left:28%;top:13%}
#rocket{width:105px;height:130px;right:12%;top:9%}
#planet{width:100px;height:70px;right:30%;top:15%}
#cat{width:130px;height:100px;left:10%;bottom:12%}
#tree{width:95px;height:160px;right:8%;bottom:10%}
#butterfly{width:100px;height:85px;left:52%;bottom:19%}
#flower{width:95px;height:105px;left:37%;bottom:7%}
#star1{width:55px;height:55px;left:49%;top:8%}
#star2{width:45px;height:45px;right:5%;top:42%}
#boat{width:110px;height:100px;left:28%;bottom:7%}
#mountain{width:160px;height:120px;right:28%;bottom:9%}
.playground-tools{display:flex;justify-content:center;flex-wrap:wrap;gap:10px;margin:18px 0 0}
.toolbtn{border:2px solid #2b2924;background:#fffdf7;font-size:17px;box-shadow:2px 3px 0 #28251f;border-radius:14px}
.toolbtn:hover{background:#fff0c1}
.lower{display:grid;grid-template-columns:1.15fr .85fr;gap:22px;margin-top:25px}
.panel{border:2px solid var(--ink);border-radius:24px;padding:20px;background:#fffdf7;box-shadow:4px 5px 0 #28251f}
.panel h2{font-size:28px;font-weight:400;margin:0 0 7px}
.panel p{font-size:18px;margin:4px 0 15px;color:#655e52}
#drawCanvas{width:100%;height:220px;display:block;border:2px dashed #c9bdab;border-radius:16px;background:#fffaf0;touch-action:none;cursor:crosshair}
.colors{display:flex;gap:9px;align-items:center;flex-wrap:wrap;margin-top:12px}
.color{width:28px;height:28px;border:2px solid var(--ink);border-radius:50%;cursor:pointer}
.color.selected{outline:3px solid #777;outline-offset:2px}
.panel-actions{display:flex;gap:9px;flex-wrap:wrap;margin-top:12px}
.note{background:#f4edff;transform:rotate(-1deg)}
.checks{list-style:none;margin:10px 0 0;padding:0;font-size:19px;line-height:1.8}
.checks label{cursor:pointer}.checks input{accent-color:#7dca9a;transform:scale(1.15);margin-right:8px}
footer{text-align:center;padding:24px;font-size:17px;color:#776c5b}
.toast{position:fixed;bottom:20px;left:50%;transform:translate(-50%,20px);opacity:0;pointer-events:none;background:#28251f;color:white;padding:10px 18px;border-radius:999px;font-size:18px;z-index:20;transition:.25s}
.toast.show{opacity:1;transform:translate(-50%,0)}
.sparkle{animation:wobble 2.8s ease-in-out infinite;transform-origin:center}
@keyframes wobble{0%,100%{transform:rotate(-5deg) translateY(0)}50%{transform:rotate(6deg) translateY(-7px)}}
@media(max-width:720px){header{flex-wrap:wrap;justify-content:center;padding:10px} .header-right{display:none}nav{order:3;width:100%;justify-content:center}nav button{font-size:16px;padding:6px 10px}.hero{padding-top:18px}.doodle-field{min-height:570px}#sun{left:5%;top:12%;width:76px;height:76px}#cloud{left:32%;top:12%;width:100px;height:70px}#rocket{right:7%;top:8%;width:75px;height:95px}#planet{right:30%;top:32%;width:75px}#cat{left:6%;bottom:15%;width:100px}#tree{right:5%;bottom:10%;width:68px;height:125px}#butterfly{left:44%;bottom:27%;width:75px}#flower{left:38%;bottom:7%;width:68px}#boat{left:7%;bottom:3%;width:85px}#mountain{right:21%;bottom:6%;width:120px}#star1{left:55%;top:8%}.lower{grid-template-columns:1fr}.panel{padding:16px}}
@media(prefers-reduced-motion:reduce){*,*:before,*:after{animation:none!important;transition:none!important}}
</style>
</head>
<body>
<header>
  <div class="brand"><span class="star">★</span> Doodle World</div>
  <nav aria-label="Main navigation">
    <button class="active" data-scroll="home">Home</button><button data-scroll="play">Doodles</button><button data-scroll="about">About</button>
  </nav>
  <div class="header-right">Click. Drag. Draw. Explore. ♡</div>
</header>
<main>
<section class="hero" id="home">
  <div class="kicker">A tiny corner of the internet ✦</div>
  <h1>Welcome to<br>Doodle World!</h1>
  <p>A playful little place where you can move the doodles, make a mess, and let your imagination wander.</p>
  <button class="primary" id="start">Start exploring&nbsp; →</button>
</section>
<section class="doodle-field" id="play" aria-label="Interactive doodle playground">
  <div class="hint">↗ Try dragging the doodles around!</div>
  <div class="doodle sparkle" id="sun" data-name="Sunny"><svg viewBox="0 0 100 100"><g stroke="#e9a928" stroke-width="4" stroke-linecap="round"><path d="M50 3v12M50 85v12M3 50h12M85 50h12M17 17l9 9M74 74l9 9M17 83l9-9M74 26l9-9"/></g><circle cx="50" cy="50" r="31" fill="#ffd45c" stroke="#28251f" stroke-width="3"/><circle cx="40" cy="46" r="2.8"/><circle cx="60" cy="46" r="2.8"/><path d="M40 58q10 10 20 0" fill="none" stroke="#28251f" stroke-width="3" stroke-linecap="round"/><circle cx="33" cy="54" r="4" fill="#ff9e88"/><circle cx="67" cy="54" r="4" fill="#ff9e88"/></svg><span class="label">Drag the sun!</span></div>
  <div class="doodle sparkle" id="cloud" data-name="Cloudy"><svg viewBox="0 0 130 90"><path d="M27 68C7 68 4 43 22 35c3-23 31-30 45-12 19-9 40 6 37 24 22 11 12 31-8 31H28" fill="#fff" stroke="#28251f" stroke-width="3.5" stroke-linejoin="round"/><circle cx="48" cy="47" r="3" fill="#28251f"/><circle cx="72" cy="47" r="3" fill="#28251f"/><path d="M51 57q9 8 18 0" fill="none" stroke="#28251f" stroke-width="2.5" stroke-linecap="round"/><circle cx="39" cy="55" r="5" fill="#ffb9b9"/><circle cx="81" cy="55" r="5" fill="#ffb9b9"/></svg></div>
  <div class="doodle sparkle" id="rocket" data-name="Rocket"><svg viewBox="0 0 100 130"><path d="M50 8C30 29 28 54 35 78l15 17 15-17C72 54 70 29 50 8Z" fill="#f7fbff" stroke="#28251f" stroke-width="3.5"/><path d="M35 63L15 80l7 16 19-11M65 63l20 17-7 16-19-11" fill="#ff7f68" stroke="#28251f" stroke-width="3"/><circle cx="50" cy="49" r="10" fill="#8dd8f4" stroke="#28251f" stroke-width="3"/><path d="M42 91l8 28 8-28-8 8Z" fill="#ffb73f" stroke="#28251f" stroke-width="2.5"/><path d="M45 97l5 17 5-17" fill="#ff6f61"/></svg><span class="label">Click to launch!</span></div>
  <div class="doodle sparkle" id="planet" data-name="Planet"><svg viewBox="0 0 110 75"><ellipse cx="55" cy="39" rx="49" ry="13" fill="#ffda72" stroke="#28251f" stroke-width="3"/><circle cx="55" cy="38" r="25" fill="#b7a2ff" stroke="#28251f" stroke-width="3"/><path d="M33 31q15 8 29 0M39 49q13 6 25-1" fill="none" stroke="#8b77db" stroke-width="3"/><path d="M10 38q40 16 90 0" fill="none" stroke="#28251f" stroke-width="3"/></svg></div>
  <div class="doodle sparkle" id="cat" data-name="Kitty"><svg viewBox="0 0 140 100"><path d="M24 58L20 20l30 18q22-9 41 5l20-18 1 39q8 25-25 29H48Q22 89 24 58Z" fill="#fff" stroke="#28251f" stroke-width="3.5" stroke-linejoin="round"/><path d="M23 34l1-14 18 11M96 32l13-8 1 17" fill="#ffb26e" stroke="#28251f" stroke-width="3"/><path d="M28 45q18-12 29 0v35q-28 3-30-15M80 43q12-7 20 3l8 25q-15 9-29 1" fill="#f6ad68"/><circle cx="52" cy="57" r="3"/><circle cx="79" cy="57" r="3"/><path d="M62 64l5 4 5-4M67 68q-7 9-14 1M67 68q7 9 14 1" fill="none" stroke="#28251f" stroke-width="2.5" stroke-linecap="round"/><circle cx="48" cy="66" r="4" fill="#ffb5bd"/><circle cx="86" cy="66" r="4" fill="#ffb5bd"/><path d="M35 63l-17-3m17 9-16 4M96 63l17-3m-17 9 16 4" stroke="#28251f" stroke-width="1.5"/></svg><span class="label">Pet the cat ♡</span></div>
  <div class="doodle sparkle" id="tree" data-name="Tree"><svg viewBox="0 0 90 160"><path d="M38 86h17l5 68H31Z" fill="#b98250" stroke="#28251f" stroke-width="3"/><path d="M45 5C25 5 19 24 23 36 4 48 12 71 25 74c-4 19 15 29 28 18 18 5 31-12 22-28 19-14 7-35-7-36C71 8 55 1 45 5Z" fill="#a9df8d" stroke="#28251f" stroke-width="3"/><circle cx="37" cy="48" r="2.8"/><circle cx="56" cy="48" r="2.8"/><path d="M39 59q7 8 15 0" fill="none" stroke="#28251f" stroke-width="2.5" stroke-linecap="round"/><path d="M27 31q8-8 14-3M57 27q8 0 11 8" fill="none" stroke="#72b65d" stroke-width="3"/></svg><span class="label">Give it a wiggle</span></div>
  <div class="doodle sparkle" id="butterfly" data-name="Butterfly"><svg viewBox="0 0 100 85"><path d="M48 42C20 2 2 20 12 42c-12 25 12 38 37 9M52 42C80 2 98 20 88 42c12 25-12 38-37 9" fill="#ffb8c7" stroke="#28251f" stroke-width="3"/><path d="M50 33q-9-15-3-19M52 33q12-15 7-19" fill="none" stroke="#28251f" stroke-width="2.5" stroke-linecap="round"/><ellipse cx="50" cy="46" rx="5" ry="17" fill="#ffd45c" stroke="#28251f" stroke-width="2.5"/><circle cx="29" cy="36" r="5" fill="#f78da9"/><circle cx="72" cy="36" r="5" fill="#f78da9"/></svg></div>
  <div class="doodle sparkle" id="flower" data-name="Flower"><svg viewBox="0 0 95 105"><path d="M48 45v56M48 75q-24-20-28 2 18 8 28-2M48 85q20-25 31-8-13 17-31 8" fill="#9bd28a" stroke="#28251f" stroke-width="3"/><g fill="#fff" stroke="#28251f" stroke-width="2.5"><circle cx="48" cy="24" r="13"/><circle cx="64" cy="35" r="13"/><circle cx="59" cy="52" r="13"/><circle cx="39" cy="52" r="13"/><circle cx="32" cy="35" r="13"/></g><circle cx="48" cy="38" r="12" fill="#ffd45c" stroke="#28251f" stroke-width="2.5"/><circle cx="44" cy="36" r="1.7"/><circle cx="52" cy="36" r="1.7"/></svg></div>
  <div class="doodle sparkle" id="star1" data-name="Star"><svg viewBox="0 0 60 60"><path d="M30 3l7 18 19 1-15 12 5 20-16-11-16 11 5-20L4 22l19-1Z" fill="#ffd45c" stroke="#28251f" stroke-width="3" stroke-linejoin="round"/></svg></div>
  <div class="doodle sparkle" id="star2" data-name="Star"><svg viewBox="0 0 60 60"><path d="M30 3l7 18 19 1-15 12 5 20-16-11-16 11 5-20L4 22l19-1Z" fill="#ffd45c" stroke="#28251f" stroke-width="3" stroke-linejoin="round"/></svg></div>
  <div class="doodle sparkle" id="boat" data-name="Boat"><svg viewBox="0 0 110 100"><path d="M54 12v63M54 17L20 63h34M60 28l29 35H60" fill="#fff" stroke="#28251f" stroke-width="3"/><path d="M10 67h91L85 87H27Z" fill="#ff8b6f" stroke="#28251f" stroke-width="3" stroke-linejoin="round"/><path d="M54 12l-8 9h16Z" fill="#ffd45c" stroke="#28251f" stroke-width="2"/></svg></div>
  <div class="doodle sparkle" id="mountain" data-name="Mountain"><svg viewBox="0 0 160 120"><path d="M5 111L54 20l22 39 24-46 55 98Z" fill="#9dd6e9" stroke="#28251f" stroke-width="3" stroke-linejoin="round"/><path d="M40 46l14-26 13 23-12-5-8 10M83 37l17-24 18 33-18-9-8 10" fill="#fff" stroke="#28251f" stroke-width="2.5" stroke-linejoin="round"/><path d="M18 98l8-5m14 10 7-4m43 1 8-5m20 11 7-4" stroke="#64b7d3" stroke-width="3" stroke-linecap="round"/></svg></div>
</section>
<div class="playground-tools">
  <button class="toolbtn" id="scatter">✨ Mix it up</button>
  <button class="toolbtn" id="party">🎉 Party mode</button>
  <button class="toolbtn" id="reset">↺ Reset doodles</button>
  <button class="toolbtn" id="surprise">🎁 Surprise me</button>
</div>
<section class="lower" id="about">
  <div class="panel">
    <h2>✏️ Your little drawing pad</h2>
    <p>Pick a color and draw absolutely anything. There are no wrong doodles.</p>
    <canvas id="drawCanvas" aria-label="Drawing area"></canvas>
    <div class="colors" aria-label="Choose drawing color">
      <button class="color selected" data-color="#25231f" style="background:#25231f" aria-label="Black"></button>
      <button class="color" data-color="#ff7d70" style="background:#ff7d70" aria-label="Coral"></button>
      <button class="color" data-color="#f5c542" style="background:#f5c542" aria-label="Yellow"></button>
      <button class="color" data-color="#7ac9e4" style="background:#7ac9e4" aria-label="Blue"></button>
      <button class="color" data-color="#a6d88b" style="background:#a6d88b" aria-label="Green"></button>
      <button class="color" data-color="#c4a8f5" style="background:#c4a8f5" aria-label="Purple"></button>
      <button class="color" data-color="#f3a8c4" style="background:#f3a8c4" aria-label="Pink"></button>
    </div>
    <div class="panel-actions"><button class="toolbtn" id="undo">↶ Undo</button><button class="toolbtn" id="clearCanvas">🧽 Clear pad</button><button class="toolbtn" id="save">⬇ Save drawing</button></div>
  </div>
  <div class="panel note">
    <h2>🌼 Tiny missions</h2>
    <p>Little ideas for your next minute of fun.</p>
    <ul class="checks">
      <li><label><input type="checkbox"> Make the sun do a dance</label></li>
      <li><label><input type="checkbox"> Give the cat a new spot</label></li>
      <li><label><input type="checkbox"> Launch the rocket</label></li>
      <li><label><input type="checkbox"> Draw a brand-new creature</label></li>
      <li><label><input type="checkbox"> Tell someone they're awesome ♡</label></li>
    </ul>
    <div style="font-size:48px;text-align:center;margin-top:8px" class="sparkle">✿ ★ ♡ ✧</div>
  </div>
</section>
<footer>Made with a little imagination and lots of ✨ · Doodle World</footer>
</main>
<div class="toast" id="toast" role="status" aria-live="polite"></div>
<script>
const field=document.querySelector('.doodle-field'), toast=document.getElementById('toast');
let toastTimer;
function say(message){toast.textContent=message;toast.classList.add('show');clearTimeout(toastTimer);toastTimer=setTimeout(()=>toast.classList.remove('show'),1900)}
const doodles=[...document.querySelectorAll('.doodle')];
const initial=new Map(doodles.map(el=>[el.id,{left:el.style.left,top:el.style.top,right:el.style.right,bottom:el.style.bottom,transform:el.style.transform}]));
doodles.forEach(el=>{
 let dragging=false,startX=0,startY=0,origX=0,origY=0;
 el.addEventListener('pointerdown',e=>{
  dragging=true;el.setPointerCapture(e.pointerId);startX=e.clientX;startY=e.clientY;
  const r=el.getBoundingClientRect(),f=field.getBoundingClientRect();origX=r.left-f.left;origY=r.top-f.top;
  el.style.left=origX+'px';el.style.top=origY+'px';el.style.right='auto';el.style.bottom='auto';el.style.zIndex=5;el.style.animation='none';
 });
 el.addEventListener('pointermove',e=>{if(!dragging)return;const f=field.getBoundingClientRect();const maxX=f.width-el.offsetWidth,maxY=f.height-el.offsetHeight;el.style.left=Math.max(0,Math.min(maxX,origX+e.clientX-startX))+'px';el.style.top=Math.max(0,Math.min(maxY,origY+e.clientY-startY))+'px';});
 el.addEventListener('pointerup',()=>{if(dragging){dragging=false;el.style.zIndex='';}});
 el.addEventListener('click',e=>{
  if(el.id==='rocket'){el.animate([{transform:'translateY(0) rotate(0)'},{transform:'translateY(-140px) rotate(18deg)',opacity:.4},{transform:'translateY(0) rotate(0)',opacity:1}],{duration:850,easing:'ease-in-out'});say('3… 2… 1… blast off! 🚀')}
  else if(el.id==='cat'){el.animate([{transform:'rotate(0) scale(1)'},{transform:'rotate(-10deg) scale(1.12)'},{transform:'rotate(10deg) scale(1.12)'},{transform:'rotate(0) scale(1)'}],{duration:600});say('Prrrr! Kitty loves you. 🐱')}
  else if(el.id==='sun'){el.animate([{transform:'rotate(0)'},{transform:'rotate(180deg)'},{transform:'rotate(360deg)'}],{duration:700});say('You make the day brighter! ☀️')}
  else if(el.id==='tree'){el.animate([{transform:'rotate(0)'},{transform:'rotate(9deg)'},{transform:'rotate(-9deg)'},{transform:'rotate(0)'}],{duration:650});say('Rustle, rustle! 🌳')}
  else if(el.id==='butterfly'){el.animate([{transform:'translate(0,0)'},{transform:'translate(35px,-25px) rotate(15deg)'},{transform:'translate(-20px,-45px) rotate(-10deg)'},{transform:'translate(0,0)'}],{duration:1000});say('Flutter flutter! 🦋')}
  else if(!e.defaultPrevented)say(el.dataset.name+' says hello! ✨');
 });
});
document.getElementById('start').addEventListener('click',()=>{document.getElementById('play').scrollIntoView({behavior:'smooth',block:'center'});say('Your playground awaits! ✨')});
document.querySelectorAll('nav button').forEach(b=>b.addEventListener('click',()=>{document.querySelectorAll('nav button').forEach(x=>x.classList.remove('active'));b.classList.add('active');document.getElementById(b.dataset.scroll).scrollIntoView({behavior:'smooth'});}));
function scatter(){const f=field.getBoundingClientRect();doodles.forEach((el,i)=>{el.style.right='auto';el.style.bottom='auto';el.style.left=Math.max(0,Math.random()*(f.width-el.offsetWidth))+'px';el.style.top=Math.max(30,Math.random()*(f.height-el.offsetHeight-10))+'px';el.style.transform=`rotate(${Math.round(Math.random()*24-12)}deg)`;});say('A fresh doodle arrangement! ✨')}
document.getElementById('scatter').addEventListener('click',scatter);
let party=false;document.getElementById('party').addEventListener('click',()=>{party=!party;document.body.style.background=party?'#fff0f5':'var(--paper)';doodles.forEach((el,i)=>{el.style.animation=party?`wobble ${.6+Math.random()*1.4}s ease-in-out infinite`:''});say(party?'Party mode ON! 🎉':'Back to chill doodles 🌼')});
document.getElementById('reset').addEventListener('click',()=>{doodles.forEach(el=>{const p=initial.get(el.id);Object.assign(el.style,{left:p.left,top:p.top,right:p.right,bottom:p.bottom,transform:p.transform,animation:''})});document.body.style.background='var(--paper)';party=false;say('Back to the original doodles!')});
const surprises=['You are a tiny star in a huge sky. ⭐','Reminder: take a stretch break! 🌱','Today is a good day to make something silly.','Sending you a pocket-sized hug. ♡','Your ideas deserve some space to grow. 🌼','You are doing better than you think.'];
document.getElementById('surprise').addEventListener('click',()=>say(surprises[Math.floor(Math.random()*surprises.length)]));
const canvas=document.getElementById('drawCanvas'),ctx=canvas.getContext('2d');let drawing=false,color='#25231f',strokes=[],current=null;
function fitCanvas(preserve=true){let old;if(preserve&&canvas.width&&canvas.height)old=canvas.toDataURL();const rect=canvas.getBoundingClientRect(),dpr=window.devicePixelRatio||1;canvas.width=Math.round(rect.width*dpr);canvas.height=Math.round(rect.height*dpr);ctx.setTransform(dpr,0,0,dpr,0,0);ctx.lineCap='round';ctx.lineJoin='round';ctx.lineWidth=4;if(old){const img=new Image();img.onload=()=>{ctx.drawImage(img,0,0,rect.width,rect.height)};img.src=old}else redraw()}
function redraw(){const dpr=window.devicePixelRatio||1;ctx.clearRect(0,0,canvas.width/dpr,canvas.height/dpr);strokes.forEach(s=>drawStroke(s));if(current)drawStroke(current)}
function drawStroke(s){if(!s.points.length)return;ctx.strokeStyle=s.color;ctx.lineWidth=s.width||4;ctx.beginPath();ctx.moveTo(s.points[0].x,s.points[0].y);s.points.slice(1).forEach(p=>ctx.lineTo(p.x,p.y));if(s.points.length===1){ctx.lineTo(s.points[0].x+.1,s.points[0].y+.1)}ctx.stroke()}
function pos(e){const r=canvas.getBoundingClientRect();return{x:e.clientX-r.left,y:e.clientY-r.top}}
canvas.addEventListener('pointerdown',e=>{drawing=true;canvas.setPointerCapture(e.pointerId);current={color,points:[pos(e)],width:4};redraw()});
canvas.addEventListener('pointermove',e=>{if(!drawing)return;current.points.push(pos(e));redraw()});
function endDraw(){if(!drawing)return;drawing=false;if(current)strokes.push(current);current=null;redraw()}
canvas.addEventListener('pointerup',endDraw);canvas.addEventListener('pointercancel',endDraw);
document.querySelectorAll('.color').forEach(b=>b.addEventListener('click',()=>{color=b.dataset.color;document.querySelectorAll('.color').forEach(x=>x.classList.toggle('selected',x===b))}));
document.getElementById('clearCanvas').addEventListener('click',()=>{strokes=[];current=null;redraw();say('Fresh page! Draw something new.')});
document.getElementById('undo').addEventListener('click',()=>{strokes.pop();redraw();say('Last stroke undone.')});
document.getElementById('save').addEventListener('click',()=>{const link=document.createElement('a');link.download='my-doodle.png';link.href=canvas.toDataURL('image/png');link.click();say('Your doodle is saved! ⬇')});
new ResizeObserver(()=>fitCanvas()).observe(canvas);fitCanvas(false);
</script>
</body>
</html>'''

(out / "index.html").write_text(html, encoding="utf-8")
readme = """# Doodle World

A small interactive doodle playground built with plain HTML, CSS, and JavaScript.

## Run it locally
1. Extract the ZIP file.
2. Open the `doodle-world` folder.
3. Double-click `index.html` to open it in your browser.

No build tools or dependencies are required. Google Fonts are optional; the page falls back to a system handwriting font.

## Publish with GitHub Pages
1. Create a new repository on GitHub.
2. Upload `index.html` (and this README if you want).
3. Open **Settings → Pages**.
4. Under **Build and deployment**, choose *from pathlib import Path
import zipfile, textwrap, os

out = Path("/mnt/data/doodle-world")
out.mkdir(exist_ok=True)

html = r'''<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Doodle World — click, drag & play</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Patrick+Hand:wght@400&family=Kalam:wght@400;700&display=swap');
:root{--paper:#fff9ed;--ink:#24231f;--yellow:#ffd15b;--pink:#ffb9c7;--blue:#b9e8f5;--green:#b8e5a0;--purple:#d9c7ff}
*{box-sizing:border-box}
body{margin:0;background:var(--paper);color:var(--ink);font-family:'Patrick Hand','Comic Sans MS',cursive;overflow-x:hidden}
header{position:sticky;top:0;z-index:10;background:rgba(255,249,237,.92);backdrop-filter:blur(10px);border-bottom:2px solid #28251f22;display:flex;align-items:center;justify-content:space-between;padding:13px clamp(16px,4vw,52px);gap:16px}
.brand{font-size:clamp(22px,3vw,32px);font-weight:bold;display:flex;align-items:center;gap:10px;white-space:nowrap}
.brand .star{font-size:30px;color:#f7bb20;text-shadow:1px 2px #222}
nav{display:flex;gap:8px;align-items:center}
nav button,.toolbtn{font:inherit;font-size:18px;border:0;background:transparent;padding:8px 13px;border-radius:999px;cursor:pointer;color:inherit}
nav button:hover,nav button.active{background:#ffdc7c}
.header-right{font-size:17px;white-space:nowrap}
main{max-width:1450px;margin:auto;padding:26px clamp(12px,3vw,40px) 55px}
.hero{text-align:center;position:relative;padding:28px 12px 18px;min-height:260px}
.kicker{font-size:18px;letter-spacing:1px;color:#686052}
h1{font-size:clamp(42px,6.5vw,76px);line-height:1.02;margin:10px 0 14px;font-weight:400}
.hero p{font-size:clamp(18px,2.2vw,23px);max-width:580px;margin:0 auto 23px;line-height:1.35}
.primary{font:inherit;font-size:21px;border:2px solid var(--ink);background:var(--yellow);padding:12px 24px;border-radius:999px;box-shadow:3px 4px 0 var(--ink);cursor:pointer;transition:transform .15s,box-shadow .15s}
.primary:hover{transform:translate(2px,2px);box-shadow:1px 2px 0 var(--ink)}
.doodle-field{position:relative;min-height:540px;border:2px dashed #cfc4ad;border-radius:28px;overflow:hidden;background:radial-gradient(#e8ddc9 1px,transparent 1px);background-size:19px 19px}
.hint{position:absolute;top:14px;left:18px;font-size:17px;color:#7c705e;z-index:2}
.doodle{position:absolute;display:grid;place-items:center;cursor:grab;user-select:none;touch-action:none;transition:filter .15s,transform .15s;filter:drop-shadow(1px 2px 0 #26231c22)}
.doodle:active{cursor:grabbing}
.doodle:hover{filter:drop-shadow(2px 4px 0 #26231c28)}
.doodle svg{width:100%;height:100%;overflow:visible}
.doodle .label{position:absolute;top:100%;font-size:16px;white-space:nowrap;transform:rotate(-5deg);pointer-events:none}
#sun{width:105px;height:105px;left:7%;top:10%}
#cloud{width:135px;height:90px;left:28%;top:13%}
#rocket{width:105px;height:130px;right:12%;top:9%}
#planet{width:100px;height:70px;right:30%;top:15%}
#cat{width:130px;height:100px;left:10%;bottom:12%}
#tree{width:95px;height:160px;right:8%;bottom:10%}
#butterfly{width:100px;height:85px;left:52%;bottom:19%}
#flower{width:95px;height:105px;left:37%;bottom:7%}
#star1{width:55px;height:55px;left:49%;top:8%}
#star2{width:45px;height:45px;right:5%;top:42%}
#boat{width:110px;height:100px;left:28%;bottom:7%}
#mountain{width:160px;height:120px;right:28%;bottom:9%}
.playground-tools{display:flex;justify-content:center;flex-wrap:wrap;gap:10px;margin:18px 0 0}
.toolbtn{border:2px solid #2b2924;background:#fffdf7;font-size:17px;box-shadow:2px 3px 0 #28251f;border-radius:14px}
.toolbtn:hover{background:#fff0c1}
.lower{display:grid;grid-template-columns:1.15fr .85fr;gap:22px;margin-top:25px}
.panel{border:2px solid var(--ink);border-radius:24px;padding:20px;background:#fffdf7;box-shadow:4px 5px 0 #28251f}
.panel h2{font-size:28px;font-weight:400;margin:0 0 7px}
.panel p{font-size:18px;margin:4px 0 15px;color:#655e52}
#drawCanvas{width:100%;height:220px;display:block;border:2px dashed #c9bdab;border-radius:16px;background:#fffaf0;touch-action:none;cursor:crosshair}
.colors{display:flex;gap:9px;align-items:center;flex-wrap:wrap;margin-top:12px}
.color{width:28px;height:28px;border:2px solid var(--ink);border-radius:50%;cursor:pointer}
.color.selected{outline:3px solid #777;outline-offset:2px}
.panel-actions{display:flex;gap:9px;flex-wrap:wrap;margin-top:12px}
.note{background:#f4edff;transform:rotate(-1deg)}
.checks{list-style:none;margin:10px 0 0;padding:0;font-size:19px;line-height:1.8}
.checks label{cursor:pointer}.checks input{accent-color:#7dca9a;transform:scale(1.15);margin-right:8px}
footer{text-align:center;padding:24px;font-size:17px;color:#776c5b}
.toast{position:fixed;bottom:20px;left:50%;transform:translate(-50%,20px);opacity:0;pointer-events:none;background:#28251f;color:white;padding:10px 18px;border-radius:999px;font-size:18px;z-index:20;transition:.25s}
.toast.show{opacity:1;transform:translate(-50%,0)}
.sparkle{animation:wobble 2.8s ease-in-out infinite;transform-origin:center}
@keyframes wobble{0%,100%{transform:rotate(-5deg) translateY(0)}50%{transform:rotate(6deg) translateY(-7px)}}
@media(max-width:720px){header{flex-wrap:wrap;justify-content:center;padding:10px} .header-right{display:none}nav{order:3;width:100%;justify-content:center}nav button{font-size:16px;padding:6px 10px}.hero{padding-top:18px}.doodle-field{min-height:570px}#sun{left:5%;top:12%;width:76px;height:76px}#cloud{left:32%;top:12%;width:100px;height:70px}#rocket{right:7%;top:8%;width:75px;height:95px}#planet{right:30%;top:32%;width:75px}#cat{left:6%;bottom:15%;width:100px}#tree{right:5%;bottom:10%;width:68px;height:125px}#butterfly{left:44%;bottom:27%;width:75px}#flower{left:38%;bottom:7%;width:68px}#boat{left:7%;bottom:3%;width:85px}#mountain{right:21%;bottom:6%;width:120px}#star1{left:55%;top:8%}.lower{grid-template-columns:1fr}.panel{padding:16px}}
@media(prefers-reduced-motion:reduce){*,*:before,*:after{animation:none!important;transition:none!important}}
</style>
</head>
<body>
<header>
  <div class="brand"><span class="star">★</span> Doodle World</div>
  <nav aria-label="Main navigation">
    <button class="active" data-scroll="home">Home</button><button data-scroll="play">Doodles</button><button data-scroll="about">About</button>
  </nav>
  <div class="header-right">Click. Drag. Draw. Explore. ♡</div>
</header>
<main>
<section class="hero" id="home">
  <div class="kicker">A tiny corner of the internet ✦</div>
  <h1>Welcome to<br>Doodle World!</h1>
  <p>A playful little place where you can move the doodles, make a mess, and let your imagination wander.</p>
  <button class="primary" id="start">Start exploring&nbsp; →</button>
</section>
<section class="doodle-field" id="play" aria-label="Interactive doodle playground">
  <div class="hint">↗ Try dragging the doodles around!</div>
  <div class="doodle sparkle" id="sun" data-name="Sunny"><svg viewBox="0 0 100 100"><g stroke="#e9a928" stroke-width="4" stroke-linecap="round"><path d="M50 3v12M50 85v12M3 50h12M85 50h12M17 17l9 9M74 74l9 9M17 83l9-9M74 26l9-9"/></g><circle cx="50" cy="50" r="31" fill="#ffd45c" stroke="#28251f" stroke-width="3"/><circle cx="40" cy="46" r="2.8"/><circle cx="60" cy="46" r="2.8"/><path d="M40 58q10 10 20 0" fill="none" stroke="#28251f" stroke-width="3" stroke-linecap="round"/><circle cx="33" cy="54" r="4" fill="#ff9e88"/><circle cx="67" cy="54" r="4" fill="#ff9e88"/></svg><span class="label">Drag the sun!</span></div>
  <div class="doodle sparkle" id="cloud" data-name="Cloudy"><svg viewBox="0 0 130 90"><path d="M27 68C7 68 4 43 22 35c3-23 31-30 45-12 19-9 40 6 37 24 22 11 12 31-8 31H28" fill="#fff" stroke="#28251f" stroke-width="3.5" stroke-linejoin="round"/><circle cx="48" cy="47" r="3" fill="#28251f"/><circle cx="72" cy="47" r="3" fill="#28251f"/><path d="M51 57q9 8 18 0" fill="none" stroke="#28251f" stroke-width="2.5" stroke-linecap="round"/><circle cx="39" cy="55" r="5" fill="#ffb9b9"/><circle cx="81" cy="55" r="5" fill="#ffb9b9"/></svg></div>
  <div class="doodle sparkle" id="rocket" data-name="Rocket"><svg viewBox="0 0 100 130"><path d="M50 8C30 29 28 54 35 78l15 17 15-17C72 54 70 29 50 8Z" fill="#f7fbff" stroke="#28251f" stroke-width="3.5"/><path d="M35 63L15 80l7 16 19-11M65 63l20 17-7 16-19-11" fill="#ff7f68" stroke="#28251f" stroke-width="3"/><circle cx="50" cy="49" r="10" fill="#8dd8f4" stroke="#28251f" stroke-width="3"/><path d="M42 91l8 28 8-28-8 8Z" fill="#ffb73f" stroke="#28251f" stroke-width="2.5"/><path d="M45 97l5 17 5-17" fill="#ff6f61"/></svg><span class="label">Click to launch!</span></div>
  <div class="doodle sparkle" id="planet" data-name="Planet"><svg viewBox="0 0 110 75"><ellipse cx="55" cy="39" rx="49" ry="13" fill="#ffda72" stroke="#28251f" stroke-width="3"/><circle cx="55" cy="38" r="25" fill="#b7a2ff" stroke="#28251f" stroke-width="3"/><path d="M33 31q15 8 29 0M39 49q13 6 25-1" fill="none" stroke="#8b77db" stroke-width="3"/><path d="M10 38q40 16 90 0" fill="none" stroke="#28251f" stroke-width="3"/></svg></div>
  <div class="doodle sparkle" id="cat" data-name="Kitty"><svg viewBox="0 0 140 100"><path d="M24 58L20 20l30 18q22-9 41 5l20-18 1 39q8 25-25 29H48Q22 89 24 58Z" fill="#fff" stroke="#28251f" stroke-width="3.5" stroke-linejoin="round"/><path d="M23 34l1-14 18 11M96 32l13-8 1 17" fill="#ffb26e" stroke="#28251f" stroke-width="3"/><path d="M28 45q18-12 29 0v35q-28 3-30-15M80 43q12-7 20 3l8 25q-15 9-29 1" fill="#f6ad68"/><circle cx="52" cy="57" r="3"/><circle cx="79" cy="57" r="3"/><path d="M62 64l5 4 5-4M67 68q-7 9-14 1M67 68q7 9 14 1" fill="none" stroke="#28251f" stroke-width="2.5" stroke-linecap="round"/><circle cx="48" cy="66" r="4" fill="#ffb5bd"/><circle cx="86" cy="66" r="4" fill="#ffb5bd"/><path d="M35 63l-17-3m17 9-16 4M96 63l17-3m-17 9 16 4" stroke="#28251f" stroke-width="1.5"/></svg><span class="label">Pet the cat ♡</span></div>
  <div class="doodle sparkle" id="tree" data-name="Tree"><svg viewBox="0 0 90 160"><path d="M38 86h17l5 68H31Z" fill="#b98250" stroke="#28251f" stroke-width="3"/><path d="M45 5C25 5 19 24 23 36 4 48 12 71 25 74c-4 19 15 29 28 18 18 5 31-12 22-28 19-14 7-35-7-36C71 8 55 1 45 5Z" fill="#a9df8d" stroke="#28251f" stroke-width="3"/><circle cx="37" cy="48" r="2.8"/><circle cx="56" cy="48" r="2.8"/><path d="M39 59q7 8 15 0" fill="none" stroke="#28251f" stroke-width="2.5" stroke-linecap="round"/><path d="M27 31q8-8 14-3M57 27q8 0 11 8" fill="none" stroke="#72b65d" stroke-width="3"/></svg><span class="label">Give it a wiggle</span></div>
  <div class="doodle sparkle" id="butterfly" data-name="Butterfly"><svg viewBox="0 0 100 85"><path d="M48 42C20 2 2 20 12 42c-12 25 12 38 37 9M52 42C80 2 98 20 88 42c12 25-12 38-37 9" fill="#ffb8c7" stroke="#28251f" stroke-width="3"/><path d="M50 33q-9-15-3-19M52 33q12-15 7-19" fill="none" stroke="#28251f" stroke-width="2.5" stroke-linecap="round"/><ellipse cx="50" cy="46" rx="5" ry="17" fill="#ffd45c" stroke="#28251f" stroke-width="2.5"/><circle cx="29" cy="36" r="5" fill="#f78da9"/><circle cx="72" cy="36" r="5" fill="#f78da9"/></svg></div>
  <div class="doodle sparkle" id="flower" data-name="Flower"><svg viewBox="0 0 95 105"><path d="M48 45v56M48 75q-24-20-28 2 18 8 28-2M48 85q20-25 31-8-13 17-31 8" fill="#9bd28a" stroke="#28251f" stroke-width="3"/><g fill="#fff" stroke="#28251f" stroke-width="2.5"><circle cx="48" cy="24" r="13"/><circle cx="64" cy="35" r="13"/><circle cx="59" cy="52" r="13"/><circle cx="39" cy="52" r="13"/><circle cx="32" cy="35" r="13"/></g><circle cx="48" cy="38" r="12" fill="#ffd45c" stroke="#28251f" stroke-width="2.5"/><circle cx="44" cy="36" r="1.7"/><circle cx="52" cy="36" r="1.7"/></svg></div>
  <div class="doodle sparkle" id="star1" data-name="Star"><svg viewBox="0 0 60 60"><path d="M30 3l7 18 19 1-15 12 5 20-16-11-16 11 5-20L4 22l19-1Z" fill="#ffd45c" stroke="#28251f" stroke-width="3" stroke-linejoin="round"/></svg></div>
  <div class="doodle sparkle" id="star2" data-name="Star"><svg viewBox="0 0 60 60"><path d="M30 3l7 18 19 1-15 12 5 20-16-11-16 11 5-20L4 22l19-1Z" fill="#ffd45c" stroke="#28251f" stroke-width="3" stroke-linejoin="round"/></svg></div>
  <div class="doodle sparkle" id="boat" data-name="Boat"><svg viewBox="0 0 110 100"><path d="M54 12v63M54 17L20 63h34M60 28l29 35H60" fill="#fff" stroke="#28251f" stroke-width="3"/><path d="M10 67h91L85 87H27Z" fill="#ff8b6f" stroke="#28251f" stroke-width="3" stroke-linejoin="round"/><path d="M54 12l-8 9h16Z" fill="#ffd45c" stroke="#28251f" stroke-width="2"/></svg></div>
  <div class="doodle sparkle" id="mountain" data-name="Mountain"><svg viewBox="0 0 160 120"><path d="M5 111L54 20l22 39 24-46 55 98Z" fill="#9dd6e9" stroke="#28251f" stroke-width="3" stroke-linejoin="round"/><path d="M40 46l14-26 13 23-12-5-8 10M83 37l17-24 18 33-18-9-8 10" fill="#fff" stroke="#28251f" stroke-width="2.5" stroke-linejoin="round"/><path d="M18 98l8-5m14 10 7-4m43 1 8-5m20 11 7-4" stroke="#64b7d3" stroke-width="3" stroke-linecap="round"/></svg></div>
</section>
<div class="playground-tools">
  <button class="toolbtn" id="scatter">✨ Mix it up</button>
  <button class="toolbtn" id="party">🎉 Party mode</button>
  <button class="toolbtn" id="reset">↺ Reset doodles</button>
  <button class="toolbtn" id="surprise">🎁 Surprise me</button>
</div>
<section class="lower" id="about">
  <div class="panel">
    <h2>✏️ Your little drawing pad</h2>
    <p>Pick a color and draw absolutely anything. There are no wrong doodles.</p>
    <canvas id="drawCanvas" aria-label="Drawing area"></canvas>
    <div class="colors" aria-label="Choose drawing color">
      <button class="color selected" data-color="#25231f" style="background:#25231f" aria-label="Black"></button>
      <button class="color" data-color="#ff7d70" style="background:#ff7d70" aria-label="Coral"></button>
      <button class="color" data-color="#f5c542" style="background:#f5c542" aria-label="Yellow"></button>
      <button class="color" data-color="#7ac9e4" style="background:#7ac9e4" aria-label="Blue"></button>
      <button class="color" data-color="#a6d88b" style="background:#a6d88b" aria-label="Green"></button>
      <button class="color" data-color="#c4a8f5" style="background:#c4a8f5" aria-label="Purple"></button>
      <button class="color" data-color="#f3a8c4" style="background:#f3a8c4" aria-label="Pink"></button>
    </div>
    <div class="panel-actions"><button class="toolbtn" id="undo">↶ Undo</button><button class="toolbtn" id="clearCanvas">🧽 Clear pad</button><button class="toolbtn" id="save">⬇ Save drawing</button></div>
  </div>
  <div class="panel note">
    <h2>🌼 Tiny missions</h2>
    <p>Little ideas for your next minute of fun.</p>
    <ul class="checks">
      <li><label><input type="checkbox"> Make the sun do a dance</label></li>
      <li><label><input type="checkbox"> Give the cat a new spot</label></li>
      <li><label><input type="checkbox"> Launch the rocket</label></li>
      <li><label><input type="checkbox"> Draw a brand-new creature</label></li>
      <li><label><input type="checkbox"> Tell someone they're awesome ♡</label></li>
    </ul>
    <div style="font-size:48px;text-align:center;margin-top:8px" class="sparkle">✿ ★ ♡ ✧</div>
  </div>
</section>
<footer>Made with a little imagination and lots of ✨ · Doodle World</footer>
</main>
<div class="toast" id="toast" role="status" aria-live="polite"></div>
<script>
const field=document.querySelector('.doodle-field'), toast=document.getElementById('toast');
let toastTimer;
function say(message){toast.textContent=message;toast.classList.add('show');clearTimeout(toastTimer);toastTimer=setTimeout(()=>toast.classList.remove('show'),1900)}
const doodles=[...document.querySelectorAll('.doodle')];
const initial=new Map(doodles.map(el=>[el.id,{left:el.style.left,top:el.style.top,right:el.style.right,bottom:el.style.bottom,transform:el.style.transform}]));
doodles.forEach(el=>{
 let dragging=false,startX=0,startY=0,origX=0,origY=0;
 el.addEventListener('pointerdown',e=>{
  dragging=true;el.setPointerCapture(e.pointerId);startX=e.clientX;startY=e.clientY;
  const r=el.getBoundingClientRect(),f=field.getBoundingClientRect();origX=r.left-f.left;origY=r.top-f.top;
  el.style.left=origX+'px';el.style.top=origY+'px';el.style.right='auto';el.style.bottom='auto';el.style.zIndex=5;el.style.animation='none';
 });
 el.addEventListener('pointermove',e=>{if(!dragging)return;const f=field.getBoundingClientRect();const maxX=f.width-el.offsetWidth,maxY=f.height-el.offsetHeight;el.style.left=Math.max(0,Math.min(maxX,origX+e.clientX-startX))+'px';el.style.top=Math.max(0,Math.min(maxY,origY+e.clientY-startY))+'px';});
 el.addEventListener('pointerup',()=>{if(dragging){dragging=false;el.style.zIndex='';}});
 el.addEventListener('click',e=>{
  if(el.id==='rocket'){el.animate([{transform:'translateY(0) rotate(0)'},{transform:'translateY(-140px) rotate(18deg)',opacity:.4},{transform:'translateY(0) rotate(0)',opacity:1}],{duration:850,easing:'ease-in-out'});say('3… 2… 1… blast off! 🚀')}
  else if(el.id==='cat'){el.animate([{transform:'rotate(0) scale(1)'},{transform:'rotate(-10deg) scale(1.12)'},{transform:'rotate(10deg) scale(1.12)'},{transform:'rotate(0) scale(1)'}],{duration:600});say('Prrrr! Kitty loves you. 🐱')}
  else if(el.id==='sun'){el.animate([{transform:'rotate(0)'},{transform:'rotate(180deg)'},{transform:'rotate(360deg)'}],{duration:700});say('You make the day brighter! ☀️')}
  else if(el.id==='tree'){el.animate([{transform:'rotate(0)'},{transform:'rotate(9deg)'},{transform:'rotate(-9deg)'},{transform:'rotate(0)'}],{duration:650});say('Rustle, rustle! 🌳')}
  else if(el.id==='butterfly'){el.animate([{transform:'translate(0,0)'},{transform:'translate(35px,-25px) rotate(15deg)'},{transform:'translate(-20px,-45px) rotate(-10deg)'},{transform:'translate(0,0)'}],{duration:1000});say('Flutter flutter! 🦋')}
  else if(!e.defaultPrevented)say(el.dataset.name+' says hello! ✨');
 });
});
document.getElementById('start').addEventListener('click',()=>{document.getElementById('play').scrollIntoView({behavior:'smooth',block:'center'});say('Your playground awaits! ✨')});
document.querySelectorAll('nav button').forEach(b=>b.addEventListener('click',()=>{document.querySelectorAll('nav button').forEach(x=>x.classList.remove('active'));b.classList.add('active');document.getElementById(b.dataset.scroll).scrollIntoView({behavior:'smooth'});}));
function scatter(){const f=field.getBoundingClientRect();doodles.forEach((el,i)=>{el.style.right='auto';el.style.bottom='auto';el.style.left=Math.max(0,Math.random()*(f.width-el.offsetWidth))+'px';el.style.top=Math.max(30,Math.random()*(f.height-el.offsetHeight-10))+'px';el.style.transform=`rotate(${Math.round(Math.random()*24-12)}deg)`;});say('A fresh doodle arrangement! ✨')}
document.getElementById('scatter').addEventListener('click',scatter);
let party=false;document.getElementById('party').addEventListener('click',()=>{party=!party;document.body.style.background=party?'#fff0f5':'var(--paper)';doodles.forEach((el,i)=>{el.style.animation=party?`wobble ${.6+Math.random()*1.4}s ease-in-out infinite`:''});say(party?'Party mode ON! 🎉':'Back to chill doodles 🌼')});
document.getElementById('reset').addEventListener('click',()=>{doodles.forEach(el=>{const p=initial.get(el.id);Object.assign(el.style,{left:p.left,top:p.top,right:p.right,bottom:p.bottom,transform:p.transform,animation:''})});document.body.style.background='var(--paper)';party=false;say('Back to the original doodles!')});
const surprises=['You are a tiny star in a huge sky. ⭐','Reminder: take a stretch break! 🌱','Today is a good day to make something silly.','Sending you a pocket-sized hug. ♡','Your ideas deserve some space to grow. 🌼','You are doing better than you think.'];
document.getElementById('surprise').addEventListener('click',()=>say(surprises[Math.floor(Math.random()*surprises.length)]));
const canvas=document.getElementById('drawCanvas'),ctx=canvas.getContext('2d');let drawing=false,color='#25231f',strokes=[],current=null;
function fitCanvas(preserve=true){let old;if(preserve&&canvas.width&&canvas.height)old=canvas.toDataURL();const rect=canvas.getBoundingClientRect(),dpr=window.devicePixelRatio||1;canvas.width=Math.round(rect.width*dpr);canvas.height=Math.round(rect.height*dpr);ctx.setTransform(dpr,0,0,dpr,0,0);ctx.lineCap='round';ctx.lineJoin='round';ctx.lineWidth=4;if(old){const img=new Image();img.onload=()=>{ctx.drawImage(img,0,0,rect.width,rect.height)};img.src=old}else redraw()}
function redraw(){const dpr=window.devicePixelRatio||1;ctx.clearRect(0,0,canvas.width/dpr,canvas.height/dpr);strokes.forEach(s=>drawStroke(s));if(current)drawStroke(current)}
function drawStroke(s){if(!s.points.length)return;ctx.strokeStyle=s.color;ctx.lineWidth=s.width||4;ctx.beginPath();ctx.moveTo(s.points[0].x,s.points[0].y);s.points.slice(1).forEach(p=>ctx.lineTo(p.x,p.y));if(s.points.length===1){ctx.lineTo(s.points[0].x+.1,s.points[0].y+.1)}ctx.stroke()}
function pos(e){const r=canvas.getBoundingClientRect();return{x:e.clientX-r.left,y:e.clientY-r.top}}
canvas.addEventListener('pointerdown',e=>{drawing=true;canvas.setPointerCapture(e.pointerId);current={color,points:[pos(e)],width:4};redraw()});
canvas.addEventListener('pointermove',e=>{if(!drawing)return;current.points.push(pos(e));redraw()});
function endDraw(){if(!drawing)return;drawing=false;if(current)strokes.push(current);current=null;redraw()}
canvas.addEventListener('pointerup',endDraw);canvas.addEventListener('pointercancel',endDraw);
document.querySelectorAll('.color').forEach(b=>b.addEventListener('click',()=>{color=b.dataset.color;document.querySelectorAll('.color').forEach(x=>x.classList.toggle('selected',x===b))}));
document.getElementById('clearCanvas').addEventListener('click',()=>{strokes=[];current=null;redraw();say('Fresh page! Draw something new.')});
document.getElementById('undo').addEventListener('click',()=>{strokes.pop();redraw();say('Last stroke undone.')});
document.getElementById('save').addEventListener('click',()=>{const link=document.createElement('a');link.download='my-doodle.png';link.href=canvas.toDataURL('image/png');link.click();say('Your doodle is saved! ⬇')});
new ResizeObserver(()=>fitCanvas()).observe(canvas);fitCanvas(false);
</script>
</body>
</html>'''

(out / "index.html").write_text(html, encoding="utf-8")
readme = """# Doodle World

A small interactive doodle playground built with plain HTML, CSS, and JavaScript.

## Run it locally
1. Extract the ZIP file.
2. Open the `doodle-world` folder.
3. Double-click `index.html` to open it in your browser.

No build tools or dependencies are required. Google Fonts are optional; the page falls back to a system handwriting font.

## Publish with GitHub Pages
1. Create a new repository on GitHub.
2. Upload `index.html` (and this README if you want).
3. Open **Settings → Pages**.
4. Under **Build and deployment**, choose **Deploy from a branch**.
5. Choose the `main` branch and `/ (root)`, then save.
6. Wait for GitHub Pages to publish your site.
"""
(out / "README.md").write_text(readme, encoding="utf-8")
zip_path = Path("/mnt/data/doodle-world-website.zip")
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for p in out.iterdir():
        z.write(p, arcname=f"doodle-world/{p.name}")
print("Created:", zip_path, "and", out / "index.html")
Deploy from a branch**.
5. Choose the `main` branch and `/ (root)`, then save.
6. Wait for GitHub Pages to publish your site.
"""
(out / "README.md").write_text(readme, encoding="utf-8")
zip_path = Path("/mnt/data/doodle-world-website.zip")
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for p in out.iterdir():
        z.write(p, arcname=f"doodle-world/{p.name}")
print("Created:", zip_path, "and", out / "index.html")
