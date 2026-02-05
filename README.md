
<html lang="zh-Hant">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1"/>
<title>超級過三關</title>
<style>
:root{
  --green:#2ecc71;
  --green-dark:#1b8f4d;
  --orange:#ff8c00;
  --board-bg:#f7f7f9;

  /* ✅ 棋盤放大：用 clamp 令大屏更大、小屏不會爆版 */
  --cell-size: clamp(120px, 19vmin, 190px);
  --gap:12px;

  --hint:#43a047;
  --move:#43a047;
  --arrowPlace:#43a047;
  --arrowMove:#43a047;

  /* ✅ 棋盤下面空間（你可自行調） */
  --board-bottom-space: 300px;

  /* ✅ Y C H 字體大小比例：接近一格（0.92 ≈ 92% cell） */
  --win-letter-scale: 0.92;
}

*{box-sizing:border-box}
body{
  margin:0;
  font-family:system-ui,-apple-system,"Segoe UI",Roboto,"Noto Sans TC","Microsoft JhengHei",Arial,sans-serif;
  background:linear-gradient(180deg,#fafafa,#f0f2f5);
  display:flex;
  min-height:100vh;
  align-items:center;
  justify-content:center;
  padding:16px
}

/* ✅ 版面：左右托盤收窄、棋盤區變大 */
.app{
  width:100%;
  max-width:1280px;
  display:grid;
  gap:16px;
  align-items:start;
  grid-template-columns: 210px 1fr 210px;
  grid-template-areas:
    "header header header"
    "left board right";
}

@media(max-width:900px){
  .app{
    grid-template-columns:1fr;
    grid-template-areas:
      "header"
      "board"
      "left"
      "right";
  }
}

/* ✅ 手機版：額外再留多啲底部空間（可自行調） */
@media(max-width:900px){
  :root{ --board-bottom-space: 110px; }
}

.header{grid-area:header;display:flex;flex-direction:column;align-items:center;gap:8px}
.title-line1{margin:0;font-weight:900;letter-spacing:.8px;color:#0f5132;font-size:clamp(28px,6.4vw,64px)}
.title-line2{margin:0;font-weight:900;letter-spacing:.6px;color:var(--green);font-size:clamp(24px,5.6vw,56px)}
.title-line2::before,.title-line2::after{content:none!important;background:none!important;box-shadow:none!important}
.title-line2 a,.title-line2 .anchor,.title-line2 [class*="icon"],.title-line2 svg{display:none!important}
.header-bar{display:flex;align-items:center;gap:10px;flex-wrap:wrap}
.dot{width:14px;height:14px;border-radius:50%}
.dot.blue{background:var(--green)}
.dot.orange{background:var(--orange)}
.turn-text{font-weight:800;font-size:14px}
.btn{
  border:1px solid #ccc;background:#fff;padding:8px 12px;border-radius:10px;cursor:pointer;
  display:inline-flex;align-items:center;justify-content:center;font-size:14px;transition:.15s
}
.btn:hover{transform:translateY(-1px);box-shadow:0 3px 10px rgba(0,0,0,.08)}
.btn:active{transform:translateY(1px)}

/* 托盤 */
.tray{
  grid-area:left;
  background:#fff;border:1px solid #e6e6e6;border-radius:14px;
  padding:12px;
  box-shadow:0 6px 16px rgba(0,0,0,.06);
}
.right{grid-area:right}

/* ✅ 托盤直排 */
.tray-grid{
  display:grid;
  grid-template-columns:1fr;
  gap:12px;
}
.tray-btn{
  display:flex;flex-direction:column;align-items:center;justify-content:center;
  gap:8px;padding:10px 8px;cursor:pointer;border-radius:12px;border:1px solid #ddd;background:#fafafa;
  transition:.15s;min-height:86px;text-align:center
}
.tray-btn:hover{background:#f5f5f5;transform:translateY(-1px)}
.tray-btn.active{border-color:#888;box-shadow:0 4px 12px rgba(0,0,0,.08);background:#fff}
.mini{
  position:relative;border-radius:50%;width:40px;height:40px;
  box-shadow:0 3px 8px rgba(0,0,0,.15),inset 0 0 0 3px rgba(255,255,255,.65)
}
.mini.size-1{width:28px;height:28px}
.mini.size-2{width:34px;height:34px}
.mini.size-3{width:40px;height:40px}
.mini.blue{background:var(--green);border:2px solid var(--green-dark)}
.mini.orange{background:var(--orange);border:2px solid #d36a00}
.mini-badge{
  position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);
  color:#fff;font-weight:900;background:rgba(0,0,0,.35);
  border-radius:999px;padding:1px 6px;font-size:12px;box-shadow:0 2px 6px rgba(0,0,0,.25);user-select:none
}
.count{font-size:13px}
.count.zero{color:#d9363e;font-weight:800}

.tray-btn.glow-green .mini{
  box-shadow:0 0 0 4px rgba(67,160,71,.85),0 0 14px 2px rgba(67,160,71,.45),inset 0 0 0 3px rgba(255,255,255,.7);
  animation:movingPulse 1.1s ease-in-out infinite
}

/* ✅ 棋盤下面留多啲空間 */
.board-wrap{
  grid-area:board;
  display:flex;
  justify-content:center;
  align-items:center;
  padding-bottom: var(--board-bottom-space);
}

.board{
  position:relative;
  display:grid;
  grid-template-columns:repeat(3,var(--cell-size));
  grid-template-rows:repeat(3,var(--cell-size));
  gap:var(--gap);
  padding:var(--gap);
  background:var(--board-bg);
  border-radius:18px;
  box-shadow:0 10px 30px rgba(0,0,0,.08),inset 0 0 0 3px #ddd
}
.cell{
  position:relative;overflow:visible;
  width:var(--cell-size);height:var(--cell-size);
  background:#fff;border-radius:12px;
  box-shadow:inset 0 0 0 2px #d0d0d0;
  cursor:pointer;
  transition:box-shadow .15s ease,transform .15s ease
}
.cell-content{position:relative;width:100%;height:100%}
.cell-overlay{position:absolute;inset:0;z-index:20;pointer-events:none}

.cell.hint{box-shadow:inset 0 0 0 3px var(--hint);animation:pulseHint 1.2s ease-in-out infinite}
.cell.hint::after{content:"放置";position:absolute;bottom:8px;right:8px;background:var(--hint);color:#fff;font-size:16px;font-weight:900;letter-spacing:.5px;padding:6px 12px;border-radius:999px;box-shadow:0 4px 12px rgba(0,0,0,.15)}
@keyframes pulseHint{0%{box-shadow:inset 0 0 0 3px var(--hint),0 0 0 0 rgba(67,160,71,.35)}50%{box-shadow:inset 0 0 0 3px var(--hint),0 0 0 10px rgba(67,160,71,0)}100%{box-shadow:inset 0 0 0 3px var(--hint),0 0 0 0 rgba(67,160,71,.35)}}
.cell.hint-move{box-shadow:inset 0 0 0 3px var(--move),0 0 0 6px rgba(67,160,71,.18);animation:targetPulse 1.05s ease-in-out infinite}
.cell.hint-move::after{content:"移動";position:absolute;bottom:8px;right:8px;background:var(--move);color:#fff;font-size:16px;font-weight:900;letter-spacing:.5px;padding:6px 12px;border-radius:999px;box-shadow:0 4px 12px rgba(0,0,0,.15)}
@keyframes targetPulse{0%{transform:scale(1)}50%{transform:scale(1.02)}100%{transform:scale(1)}}
.cell.source-cue{box-shadow:inset 0 0 0 3px var(--hint),0 0 0 6px rgba(67,160,71,.18)}

.piece{
  position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);
  border-radius:50%;overflow:hidden;
  box-shadow:0 6px 16px rgba(0,0,0,.18),inset 0 0 0 3px rgba(255,255,255,.65);
  transition:transform .18s ease,filter .18s ease,box-shadow .18s ease,opacity .18s ease
}
.size-1{width:55%;height:55%}
.size-2{width:72%;height:72%}
.size-3{width:95%;height:95%}
.blue-piece{background:var(--green);border:2px solid var(--green-dark)}
.orange-piece{background:var(--orange);border:2px solid #d36a00}
.blue-piece::before{content:"";position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:72%;height:72%;border-radius:50%;box-shadow:0 0 0 5px rgba(255,255,255,.95),inset 0 0 0 7px var(--green-dark)}
.orange-piece::before,.orange-piece::after{content:"";position:absolute;left:50%;top:50%;width:76%;height:12%;background:#d36a00;border-radius:8px;transform-origin:center;box-shadow:0 0 0 4px rgba(255,255,255,.95),0 1px 2px rgba(0,0,0,.2)}
.orange-piece::before{transform:translate(-50%,-50%) rotate(45deg)}
.orange-piece::after{transform:translate(-50%,-50%) rotate(-45deg)}
.moving-piece{box-shadow:0 0 0 4px rgba(67,160,71,.85),0 0 14px 2px rgba(67,160,71,.45),inset 0 0 0 3px rgba(255,255,255,.7);animation:movingPulse 1.1s ease-in-out infinite}
@keyframes movingPulse{0%{transform:translate(-50%,-50%) scale(1)}50%{transform:translate(-50%,-50%) scale(1.03)}100%{transform:translate(-50%,-50%) scale(1)}}

.win-spotlight .board .piece{opacity:.28;filter:grayscale(.1) saturate(.8)}
.win-spotlight .board .win-pulse{opacity:1;filter:none}
.win-pulse{box-shadow:0 0 0 6px rgba(67,160,71,.95),0 0 28px 8px rgba(67,160,71,.55),inset 0 0 0 3px rgba(255,255,255,.9);animation:winGlow .85s ease-in-out infinite}
.piece.win-pulse::after{content:"";position:absolute;inset:-8%;border-radius:50%;box-shadow:0 0 0 0 rgba(67,160,71,.75);animation:winRing 1.1s ease-out infinite}
@keyframes winGlow{0%{transform:translate(-50%,-50%) scale(1);filter:saturate(1.4) brightness(1.08)}50%{transform:translate(-50%,-50%) scale(1.12);filter:saturate(1.6) brightness(1.18)}100%{transform:translate(-50%,-50%) scale(1);filter:saturate(1.4) brightness(1.08)}}
@keyframes winRing{0%{box-shadow:0 0 0 0 rgba(67,160,71,.75)}100%{box-shadow:0 0 0 22px rgba(67,160,71,0)}}
.size-badge{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);color:#fff;font-weight:900;background:rgba(0,0,0,.35);border-radius:999px;padding:2px 8px;box-shadow:0 2px 6px rgba(0,0,0,.25);user-select:none;z-index:3}

/* ✅ Y C H：放大到接近一格大小 */
.win-letter,
.win-letter-still{
  position:absolute;
  left:50%;
  top:50%;
  transform:translate(-50%,-50%);
  font-weight:1000;
  color:#16a34a;
  line-height:1;

  /* ⭐ 直接用 cell-size 計：幾乎等於一格 */
  font-size: calc(var(--cell-size) * var(--win-letter-scale));

  /* 加強可讀性：白邊 + 光暈 */
  text-shadow:
    0 2px 0 #fff,
    0 0 12px rgba(22,163,74,.55),
    0 0 22px rgba(22,163,74,.35);
  pointer-events:none;
  z-index:30;
}

/* 動畫版保持 pop 效果 */
.win-letter{
  transform:translate(-50%,-50%) scale(.2);
  opacity:0;
  animation:pop .5s ease forwards
}
@keyframes pop{
  0%{transform:translate(-50%,-50%) scale(.2);opacity:0}
  60%{transform:translate(-50%,-50%) scale(1.08);opacity:1}
  100%{transform:translate(-50%,-50%) scale(1)}
}

.arrow-layer{position:fixed;left:0;top:0;pointer-events:none;z-index:9999}
.arrow-path{fill:none;stroke-width:2.5;stroke-linecap:round;stroke-linejoin:round;stroke-dasharray:5 12;opacity:.8;animation:dashMove 1.2s linear infinite;filter:drop-shadow(0 1px 2px rgba(0,0,0,.15))}
@keyframes dashMove{to{stroke-dashoffset:-14}}

.ghost{position:fixed;left:0;top:0;transform:translate(-50%,-50%);transition:left .65s ease,top .65s ease;pointer-events:none;z-index:9000;will-change:left,top}

.msg{position:fixed;left:50%;bottom:14px;transform:translateX(-50%);background:#111;color:#fff;padding:8px 12px;border-radius:10px;font-size:13px;opacity:0;transition:opacity .2s}
.msg.show{opacity:.9}
.cell:active{transform:scale(0.985)}
.tray-btn:active{transform:translateY(0);box-shadow:0 1px 6px rgba(0,0,0,.08)}
@media (prefers-reduced-motion: reduce){*{animation:none!important;transition:none!important}.arrow-path{animation:none!important}.moving-piece,.win-pulse{animation:none!important}}
</style>
</head>

<body>
<svg id="arrowLayer" class="arrow-layer" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true" width="100%" height="100%">
  <defs>
    <marker id="headPlace" markerWidth="7" markerHeight="7" refX="5.6" refY="3.5" orient="auto">
      <polygon points="0,0 7,3.5 0,7" fill="var(--arrowPlace)"></polygon>
    </marker>
    <marker id="headMove" markerWidth="7" markerHeight="7" refX="5.6" refY="3.5" orient="auto">
      <polygon points="0,0 7,3.5 0,7" fill="var(--arrowMove)"></polygon>
    </marker>
  </defs>
  <path id="arrowPath" class="arrow-path" d="" stroke="transparent" marker-end="url(#headPlace)"></path>
</svg>

<div class="app">
  <div class="header">
    <h1 class="title-line1">仁濟STEAM FAIRE 2026</h1>
    <h2 class="title-line2">超級過三關</h2>
    <div class="header-bar">
      <span id="turnDot" class="dot blue"></span>
      <span id="turnText" class="turn-text">輪到：綠</span>
      <button id="restartBtn" class="btn" style="display:none;">重新開始</button>
      <button id="swapBtn" class="btn" style="display:none;">換邊起手</button>

      <!-- ✅ 改字眼：教學模式 -> AI對戰模式 -->
      <button id="modeBtn" class="btn">退出AI對戰模式</button>
    </div>
  </div>

  <div class="tray" id="trayBlue">
    <div class="tray-grid">
      <div class="tray-btn" data-player="blue" data-size="3">
        <div class="mini blue size-3"><span class="mini-badge">大</span></div>
        <div class="count" id="count-blue-3">x 2</div>
      </div>
      <div class="tray-btn" data-player="blue" data-size="2">
        <div class="mini blue size-2"><span class="mini-badge">中</span></div>
        <div class="count" id="count-blue-2">x 2</div>
      </div>
      <div class="tray-btn" data-player="blue" data-size="1">
        <div class="mini blue size-1"><span class="mini-badge">小</span></div>
        <div class="count" id="count-blue-1">x 2</div>
      </div>
    </div>
  </div>

  <div class="board-wrap">
    <div id="board" class="board" aria-label="3x3"></div>
  </div>

  <div class="tray right" id="trayOrange">
    <div class="tray-grid">
      <div class="tray-btn" data-player="orange" data-size="3">
        <div class="mini orange size-3"><span class="mini-badge">大</span></div>
        <div class="count" id="count-orange-3">x 2</div>
      </div>
      <div class="tray-btn" data-player="orange" data-size="2">
        <div class="mini orange size-2"><span class="mini-badge">中</span></div>
        <div class="count" id="count-orange-2">x 2</div>
      </div>
      <div class="tray-btn" data-player="orange" data-size="1">
        <div class="mini orange size-1"><span class="mini-badge">小</span></div>
        <div class="count" id="count-orange-1">x 2</div>
      </div>
    </div>
  </div>
</div>

<div id="msg" class="msg" role="status" aria-live="polite"></div>

<script>
(function(){
const sizeNames={1:"小",2:"中",3:"大"},
      winLines=[[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]],
      WIN_DELAY=3000,CONFIRM_DELAY=1000;

const boardEl=document.getElementById("board"),
      turnDot=document.getElementById("turnDot"),
      turnText=document.getElementById("turnText");
const restartBtn=document.getElementById("restartBtn"),
      swapBtn=document.getElementById("swapBtn"),
      modeBtn=document.getElementById("modeBtn");
const arrowLayer=document.getElementById('arrowLayer'),
      arrowPath=document.getElementById('arrowPath'),
      msgEl=document.getElementById('msg');

let board,counts,current,selectedSize,gameOver;
let teachingMode=true,stepIndex=0,movingFromIndex=null,pvpSelectedFrom=null;
let winLetters={},currentArrow=null,ghostAnim=null,winPulse=new Set(),winLineIdx=null;

/* －－－－ 音效引擎：更有趣的合成音（Web Audio API） －－－－ */
const audio = {
  ctx:null,
  enabled:true,
  volume:0.7,
  master:null, comp:null, delay:null, delayGain:null, limiter:null,
  ensureCtx(){
    if(!this.ctx){
      const AC=window.AudioContext||window.webkitAudioContext;
      if(!AC) return;
      this.ctx=new AC();
      this.master=this.ctx.createGain(); this.master.gain.value=this.volume;
      this.comp=this.ctx.createDynamicsCompressor();
      this.comp.threshold.setValueAtTime(-22,this.ctx.currentTime);
      this.comp.knee.setValueAtTime(30,this.ctx.currentTime);
      this.comp.ratio.setValueAtTime(6,this.ctx.currentTime);
      this.comp.attack.setValueAtTime(0.003,this.ctx.currentTime);
      this.comp.release.setValueAtTime(0.12,this.ctx.currentTime);

      this.delay=this.ctx.createDelay(0.6);
      this.delay.delayTime.value=0.18;
      this.delayGain=this.ctx.createGain(); this.delayGain.gain.value=0.18;
      this.delay.connect(this.delayGain).connect(this.delay);

      const delayOut=this.ctx.createGain(); delayOut.gain.value=0.22;
      this.delay.connect(delayOut);

      this.limiter=this.ctx.createDynamicsCompressor();
      this.limiter.threshold.setValueAtTime(-3,this.ctx.currentTime);
      this.limiter.knee.setValueAtTime(0,this.ctx.currentTime);
      this.limiter.ratio.setValueAtTime(20,this.ctx.currentTime);
      this.limiter.attack.setValueAtTime(0.002,this.ctx.currentTime);
      this.limiter.release.setValueAtTime(0.08,this.ctx.currentTime);

      this.comp.connect(this.master);
      delayOut.connect(this.master);
      this.master.connect(this.limiter).connect(this.ctx.destination);
    }
  },
  setMasterVolume(v){
    this.volume=v;
    if(this.master){
      const ct=this.ctx.currentTime;
      this.master.gain.cancelScheduledValues(ct);
      this.master.gain.linearRampToValueAtTime(v, ct+0.05);
    }
  },
  now(){ this.ensureCtx(); return this.ctx?this.ctx.currentTime:0; },
  applyADSR(g, t0, {a=0.01,d=0.08,s=0.5,r=0.06,peak=0.9}={}){
    g.gain.setValueAtTime(0.0001, t0);
    g.gain.linearRampToValueAtTime(peak, t0 + a);
    g.gain.linearRampToValueAtTime(s*peak, t0 + a + d);
    return (tRelease)=>{
      g.gain.cancelScheduledValues(tRelease);
      g.gain.setValueAtTime(g.gain.value, tRelease);
      g.gain.linearRampToValueAtTime(0.0001, tRelease + r);
    };
  },
  noise(duration=0.2){
    const len=Math.max(1,Math.floor((this.ctx.sampleRate||44100)*duration));
    const buf=this.ctx.createBuffer(1,len,this.ctx.sampleRate);
    const data=buf.getChannelData(0);
    for(let i=0;i<len;i++){ data[i]=(Math.random()*2-1)*0.4; }
    const src=this.ctx.createBufferSource(); src.buffer=buf;
    return src;
  },
  toBus(node){
    if(!this.ctx) return;
    node.connect(this.comp);
    node.connect(this.delay);
  },
  whoosh({time=this.now(), dur=0.2, pan=0}={}){
    if(!this.ctx) return;
    const src=this.noise(dur);
    const bp=this.ctx.createBiquadFilter(); bp.type='bandpass'; bp.Q.value=1.2; bp.frequency.setValueAtTime(400, time);
    const g=this.ctx.createGain(); g.gain.value=0.0;
    const rel=this.applyADSR(g, time, {a:0.015,d:0.09,s:0.0,r:0.07,peak:0.6});
    const hasPanner = 'createStereoPanner' in this.ctx;
    const p=hasPanner?this.ctx.createStereoPanner():null;
    if(p){ p.pan.value=pan; src.connect(bp).connect(g).connect(p); this.toBus(p); }
    else{ src.connect(bp).connect(g); this.toBus(g); }
    bp.frequency.linearRampToValueAtTime(2200, time + dur*0.7);
    src.start(time); rel(time + dur*0.65); src.stop(time + dur + 0.05);
  },
  click({time=this.now(), freq=1800}={}){
    if(!this.ctx) return;
    const src=this.noise(0.06);
    const hp=this.ctx.createBiquadFilter(); hp.type='highpass'; hp.frequency.setValueAtTime(600, time);
    const g=this.ctx.createGain(); g.gain.value=0.0;
    const rel=this.applyADSR(g, time, {a:0.005,d:0.03,s:0.0,r:0.02,peak:0.5});
    src.connect(hp).connect(g); this.toBus(g);
    src.start(time); rel(time+0.04); src.stop(time+0.08);
  },
  noteToFreq(note){ return 440*Math.pow(2,(note-69)/12); },
  pickPentatonic(root, octave, steps){
    const scale=[0,2,4,7,9];
    return steps.map(s=>{
      const degree=scale[(s%scale.length+scale.length)%scale.length];
      const oct = octave + Math.floor(s/scale.length);
      return root + degree + 12*oct;
    });
  },
  pluck(freq, {time=this.now(), dur=0.18, detune=0, cutoff=2200, pan=0, gain=0.8, type='triangle'}={}){
    if(!this.ctx) return;
    const osc1=this.ctx.createOscillator(); osc1.type=type; osc1.frequency.setValueAtTime(freq, time); osc1.detune.value=detune;
    const osc2=this.ctx.createOscillator(); osc2.type=type; osc2.frequency.setValueAtTime(freq*1.5, time); osc2.detune.value=-detune*0.5;
    const lp=this.ctx.createBiquadFilter(); lp.type='lowpass'; lp.frequency.setValueAtTime(cutoff, time); lp.Q.value=0.8;
    const g=this.ctx.createGain(); g.gain.value=0.0;
    const release=this.applyADSR(g, time, {a:0.005,d:0.09,s:0.0,r:0.05,peak:gain});
    const hasPanner='createStereoPanner' in this.ctx; const p=hasPanner?this.ctx.createStereoPanner():null;
    if(p){ p.pan.value=pan; osc1.connect(lp); osc2.connect(lp); lp.connect(g).connect(p); this.toBus(p); }
    else{ osc1.connect(lp); osc2.connect(lp); lp.connect(g); this.toBus(g); }
    lp.frequency.exponentialRampToValueAtTime(600, time + dur*0.8);
    osc1.start(time); osc2.start(time);
    release(time + dur*0.75);
    osc1.stop(time + dur + 0.05); osc2.stop(time + dur + 0.05);
  },
  glide(freqStart, freqEnd, {time=this.now(), dur=0.16, pan=0, gain=0.6, type='sine'}={}){
    if(!this.ctx) return;
    const osc=this.ctx.createOscillator(); osc.type=type; osc.frequency.setValueAtTime(freqStart, time);
    osc.frequency.exponentialRampToValueAtTime(Math.max(1,freqEnd), time + dur*0.9);
    const g=this.ctx.createGain(); g.gain.value=0.0;
    const release=this.applyADSR(g, time, {a:0.008,d:0.06,s:0.0,r:0.05,peak:gain});
    const hasPanner='createStereoPanner' in this.ctx; const p=hasPanner?this.ctx.createStereoPanner():null;
    if(p){ p.pan.value=pan; osc.connect(g).connect(p); this.toBus(p); }
    else{ osc.connect(g); this.toBus(g); }
    osc.start(time); release(time + dur*0.8); osc.stop(time + dur + 0.05);
  },
  chord(freqs, {time=this.now(), dur=0.22, pan=0, gain=0.6, type='square'}={}){
    if(!this.ctx) return;
    const g=this.ctx.createGain(); g.gain.value=0.0;
    const release=this.applyADSR(g, time, {a:0.01,d:0.08,s:0.15,r:0.08,peak:gain});
    const hasPanner='createStereoPanner' in this.ctx; const p=hasPanner?this.ctx.createStereoPanner():null;
    const lp=this.ctx.createBiquadFilter(); lp.type='lowpass'; lp.frequency.value=3600;
    const oscs=[];
    freqs.forEach((f,i)=>{
      const o=this.ctx.createOscillator(); o.type=type; o.frequency.setValueAtTime(f, time);
      o.detune.value= (i-1)*6; oscs.push(o);
      o.connect(lp);
    });
    if(p){ p.pan.value=pan; lp.connect(g).connect(p); this.toBus(p); }
    else{ lp.connect(g); this.toBus(g); }
    oscs.forEach(o=>o.start(time));
    release(time + dur*0.75);
    oscs.forEach(o=>o.stop(time + dur + 0.05));
  },

  sfxPlace(player, size){
    if(!teachingMode || !this.enabled) return;
    this.ensureCtx(); if(!this.ctx) return;
    const t=this.now();
    const root = (player==='blue')? 62 : 57;
    const octave = (player==='blue')? 1 : 2;
    const sizeShift = ({1:+7,2:+0,3:-5})[size]||0;
    const notes = this.pickPentatonic(root+sizeShift, octave, [0,1,2]);
    const pan = (player==='blue')? -0.15 : 0.15;
    notes.forEach((n,i)=>{
      const f=this.noteToFreq(n);
      this.pluck(f,{time:t + i*0.055, dur:0.18 - i*0.02, pan, gain:0.72, cutoff:2600, type:'triangle'});
    });
    this.click({time: t + 0.03});
    const f0 = this.noteToFreq((notes[1]||root)+12);
    const f1 = this.noteToFreq(notes[1]||root);
    this.glide(f0, f1, {time: t + 0.04, dur:0.12, pan, gain:0.22, type:'sine'});
  },
  sfxMove(player, size){
    if(!teachingMode || !this.enabled) return;
    this.ensureCtx(); if(!this.ctx) return;
    const t=this.now();
    const root = (player==='blue')? 50 : 55;
    const octave = 2;
    const pan = (player==='blue')? -0.22 : 0.22;
    const siz = ({1:+5,2:+0,3:-5})[size]||0;
    const seq = this.pickPentatonic(root+siz, octave, [1,3,2]);
    this.pluck(this.noteToFreq(seq[0]), {time:t, dur:0.16, pan, gain:0.7, cutoff:3000, type:'triangle'});
    this.pluck(this.noteToFreq(seq[1]), {time:t+0.09, dur:0.13, pan, gain:0.55, cutoff:2400, type:'triangle'});
    this.pluck(this.noteToFreq(seq[2]), {time:t+0.16, dur:0.12, pan, gain:0.52, cutoff:2200, type:'triangle'});
    this.whoosh({time:t+0.02, dur:0.22, pan});
    const fStart=this.noteToFreq(seq[0]+7);
    const fEnd=this.noteToFreq(seq[0]-5);
    this.glide(fStart, fEnd, {time:t+0.05, dur:0.18, pan, gain:0.28, type:'sine'});
  },
  sfxWin(){
    if(!teachingMode || !this.enabled) return;
    this.ensureCtx(); if(!this.ctx) return;
    const t=this.now();
    const tri=[62,66,69].map(n=>this.noteToFreq(n+12));
    this.chord(tri,{time:t, dur:0.26, pan:0, gain:0.65, type:'square'});
    const line=[69,71,74].map(n=>this.noteToFreq(n+12));
    this.pluck(line[0],{time:t+0.14, dur:0.16, pan:0.05, gain:0.66, cutoff:3600});
    this.pluck(line[1],{time:t+0.26, dur:0.14, pan:-0.05, gain:0.6, cutoff:3600});
    this.pluck(line[2],{time:t+0.36, dur:0.14, pan:0.08, gain:0.6, cutoff:3600});
    const sparkleT=t+0.38;
    const src=this.noise(0.18);
    const hp=this.ctx.createBiquadFilter(); hp.type='highpass'; hp.frequency.value=3200;
    const g=this.ctx.createGain(); g.gain.value=0;
    const rel=this.applyADSR(g, sparkleT, {a:0.005,d:0.08,s:0.0,r:0.06,peak:0.42});
    src.connect(hp).connect(g); this.toBus(g);
    src.start(sparkleT); rel(sparkleT+0.12); src.stop(sparkleT+0.2);
  }
};

/* －－－－ AI對戰腳本（原教學腳本） －－－－ */
const SCRIPT=[
  {actor:'blue',type:'place',size:3,to:4},
  {actor:'orange',type:'place',size:3,to:8},
  {actor:'blue',type:'place',size:3,to:2},
  {actor:'orange',type:'place',size:2,to:6},
  {actor:'blue',type:'move',size:3,from:2,to:6},
  {actor:'orange',type:'place',size:2,to:2},
  {actor:'blue',type:'move',size:3,from:4,to:2},
  {actor:'orange',type:'place',size:3,to:4},
  {actor:'blue',type:'place',size:1,to:0},
  {actor:'orange',type:'move',size:3,from:8,to:0},
  {actor:'blue',type:'place',size:2,to:8},
  {actor:'orange',type:'move',size:3,from:4,to:8},
  {actor:'blue',type:'place',size:2,to:4}
];

if(!boardEl.children.length){
  for(let i=0;i<9;i++){
    const c=document.createElement("div"); c.className="cell"; c.dataset.index=i;
    const content=document.createElement('div'); content.className='cell-content';
    const overlay=document.createElement('div'); overlay.className='cell-overlay';
    c.appendChild(content); c.appendChild(overlay);
    c.addEventListener("click",()=>onCellClick(i));
    boardEl.appendChild(c);
  }
}

function resetCommon(){
  board=Array.from({length:9},()=>[]);
  counts={blue:{1:2,2:2,3:2},orange:{1:2,2:2,3:2}};
  selectedSize=null; gameOver=false; movingFromIndex=null; pvpSelectedFrom=null;
  currentArrow=null; clearArrow(); winPulse.clear(); winLineIdx=null;
  render(); clearHints(); clearTrayGlow();
}
function clearWinLettersDOM(){
  Array.from(boardEl.children).forEach(c=>{
    const ov=c.querySelector('.cell-overlay'); if(ov) ov.innerHTML="";
  });
  winLetters={};
}

function resetTeaching(){
  clearWinLettersDOM();
  teachingMode=true; stepIndex=0;
  modeBtn.textContent="退出AI對戰模式";
  restartBtn.style.display="none"; swapBtn.style.display="none";
  resetCommon(); current="blue"; render(); showNextHint();
}
function resetPVP(start="blue"){
  clearWinLettersDOM();
  teachingMode=false;
  modeBtn.textContent="開始AI對戰模式";
  restartBtn.style.display=""; swapBtn.style.display="";
  resetCommon(); current=start; render();
  hint("PVP 開始，先手："+(current==="blue"?"綠":"橙"));
}

restartBtn.addEventListener("click",()=>{ if(gameOver) return; if(!teachingMode) resetPVP("blue"); });
swapBtn.addEventListener("click",()=>{ if(gameOver) return; if(!teachingMode){ current=(current==="blue")?"orange":"blue"; resetPVP(current); hint("已換邊起手："+(current==="blue"?"綠":"橙")); }});
modeBtn.addEventListener("click",()=>{ teachingMode?resetPVP("blue"):resetTeaching(); });

document.querySelectorAll(".tray-btn").forEach(btn=>{
  btn.addEventListener("click",()=>{
    if(teachingMode) return;
    if(gameOver) return;
    const player=btn.dataset.player,size=Number(btn.dataset.size);
    if(player!==current){ hint("唔到你用對家托盤"); return; }
    if(counts[player][size]<=0){ hint("此大小已用完"); return; }
    if(pvpSelectedFrom!==null){ const el=boardEl.children[pvpSelectedFrom]; el&&el.classList.remove('source-cue'); pvpSelectedFrom=null; }
    if(selectedSize===size){ selectedSize=null; clearTrayGlow(); hint("已取消選擇"); return; }
    selectedSize=size; clearTrayGlow(); btn.classList.add("glow-green","active");
    hint("已選："+(current==="blue"?"綠":"橙")+"「"+sizeNames[size]+"」");
  });
});

function layoutArrowLayer(){
  const vv=window.visualViewport;
  if(vv){
    arrowLayer.style.width=vv.width+"px"; arrowLayer.style.height=vv.height+"px";
    arrowLayer.style.left="0px"; arrowLayer.style.top="0px";
    arrowLayer.setAttribute('viewBox',`0 0 ${vv.width} ${vv.height}`);
  }else{
    arrowLayer.style.width="100vw"; arrowLayer.style.height="100vh";
    arrowLayer.style.left="0px"; arrowLayer.style.top="0px";
    arrowLayer.setAttribute('viewBox',`0 0 ${window.innerWidth} ${window.innerHeight}`);
  }
}
function getCenter(el){
  if(!el) return null;
  const r=el.getBoundingClientRect();
  return {x:r.left+r.width/2,y:r.top+r.height/2,w:r.width,h:r.height};
}
function offsetEndpoints(aEl,bEl){
  const A=getCenter(aEl),B=getCenter(bEl); if(!A||!B) return null;
  const dx=B.x-A.x,dy=B.y-A.y,len=Math.hypot(dx,dy)||1,nx=-dy/len,ny=dx/len,base=Math.min(A.w||0,B.w||0)||60,off=Math.min(22,base*0.10);
  const f={x:A.x+nx*off,y:A.y+ny*off},t={x:B.x-nx*off,y:B.y-ny*off},mid={x:(f.x+t.x)/2,y:(f.y+t.y)/2};
  return {f,t,mid,nx,ny,len};
}
function drawArrow(aEl,bEl,kind){
  if(!aEl||!bEl){ clearArrow(); return; }
  layoutArrowLayer();
  const p=offsetEndpoints(aEl,bEl); if(!p){ clearArrow(); return; }
  const bend=Math.min(28,p.len*0.10),cx=p.mid.x+p.nx*bend,cy=p.mid.y+p.ny*bend;
  arrowPath.setAttribute('d',`M ${p.f.x},${p.f.y} Q ${cx},${cy} ${p.t.x},${p.t.y}`);
  arrowPath.setAttribute('stroke',getComputedStyle(document.documentElement).getPropertyValue(kind==='place'?'--arrowPlace':'--arrowMove')||'#43a047');
  arrowPath.setAttribute('marker-end',`url(#${kind==='place'?'headPlace':'headMove'})`);
  arrowPath.style.opacity='1'; currentArrow={fromEl:aEl,toEl:bEl,kind};
}
function clearArrow(){ arrowPath.setAttribute('d',''); arrowPath.style.opacity='0'; currentArrow=null; }

function ghostMove(from,toEl,player,size,dur=650){
  return new Promise(res=>{
    const A=(from&&from.nodeType===1)?getCenter(from):from, B=getCenter(toEl);
    if(!A||!B){ res(); return; }
    const cw=B.w||80,ratio=(size===3?0.95:size===2?0.72:0.55),wh=cw*ratio;
    const g=document.createElement("div");
    g.className=`piece ${player==='blue'?'blue-piece':'orange-piece'} size-${size} ghost`;
    g.style.left=A.x+"px"; g.style.top=A.y+"px"; g.style.width=wh+"px"; g.style.height=wh+"px";
    const badge=document.createElement("span"); badge.className="size-badge"; badge.textContent=sizeNames[size]; g.appendChild(badge);
    g.style.transitionDuration=dur+"ms"; document.body.appendChild(g);
    g.getBoundingClientRect();
    requestAnimationFrame(()=>{ const B2=getCenter(toEl); if(B2){ g.style.left=B2.x+"px"; g.style.top=B2.y+"px"; } });
    const end=Date.now()+dur+30; ghostAnim={el:g,toEl,endTime:end};
    setTimeout(()=>{ ghostAnim=null; g.remove(); res(); }, dur+40);
  });
}

function topPiece(i){const s=board[i];return s.length?s[s.length-1]:null;}
function canPlace(player,size,i){const s=board[i],t=s.length?s[s.length-1]:null;return !t||size>t.size;}
function canMove(player,size,from,to){
  if(from===to)return false;
  const ft=topPiece(from);
  if(!ft||ft.player!==player||ft.size!==size)return false;
  const tt=topPiece(to);
  return !tt||size>tt.size;
}
function checkWin(p){return winLines.some(line=>line.every(i=>{const t=topPiece(i);return t&&t.player===p;}));}
function getWinningLine(p){
  for(const l of winLines){
    if(l.every(i=>{const t=topPiece(i);return t&&t.player===p;})) return l;
  }
  return null;
}

function render(){
  turnDot.className="dot "+(current==="blue"?"blue":"orange");
  turnText.textContent="輪到："+(current==="blue"?"綠":"橙");
  for(let i=0;i<9;i++){
    const cell=boardEl.children[i],content=cell.querySelector('.cell-content'),overlay=cell.querySelector('.cell-overlay');
    content.innerHTML="";
    const t=topPiece(i);
    if(t){
      const p=document.createElement("div");
      p.className=`piece ${t.player==='blue'?'blue-piece':'orange-piece'} size-${t.size}`;
      if(i===movingFromIndex)p.classList.add('moving-piece');
      if(winPulse.has(i))p.classList.add('win-pulse');
      const b=document.createElement("span"); b.className="size-badge"; b.textContent=sizeNames[t.size]; p.appendChild(b);
      content.appendChild(p);
    }
    if(winLetters[i] && !overlay.querySelector('.win-letter,.win-letter-still')){
      const s=document.createElement('span'); s.className='win-letter-still'; s.textContent=winLetters[i]; overlay.appendChild(s);
    }
  }
  [1,2,3].forEach(s=>{
    const cb=document.getElementById(`count-blue-${s}`),co=document.getElementById(`count-orange-${s}`);
    if(cb){cb.textContent=`x ${counts.blue[s]}`;cb.classList.toggle("zero",counts.blue[s]===0)}
    if(co){co.textContent=`x ${counts.orange[s]}`;co.classList.toggle("zero",counts.orange[s]===0)}
  });
}
function clearHints(){Array.from(boardEl.children).forEach(c=>c.classList.remove("hint","hint-move","source-cue"))}
function clearTrayGlow(){document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("glow-green","active"))}
function switchTurn(){current=(current==="blue")?"orange":"blue";render()}

function showNextHint(){
  clearHints(); clearTrayGlow(); movingFromIndex=null;
  if(!teachingMode||gameOver||stepIndex>=SCRIPT.length){render();clearArrow();return}
  const mv=SCRIPT[stepIndex]; current=mv.actor; render();
  if(mv.actor==='blue'){ clearArrow(); return; }
  if(mv.type==='place'){
    const dst=boardEl.children[mv.to]; dst&&dst.classList.add("hint");
    const trayBtn=[...document.querySelectorAll('#trayOrange .tray-btn')].find(b=>Number(b.dataset.size)===mv.size);
    const dot=trayBtn?trayBtn.querySelector('.mini'):trayBtn; drawArrow(dot||trayBtn,dst,'place');
  }else{
    const src=boardEl.children[mv.from],dst=boardEl.children[mv.to];
    src&&src.classList.add("source-cue"); dst&&dst.classList.add("hint-move");
    drawArrow(src,dst,'move');
  }
}

let uiLocked=false;
function lock(){ uiLocked=true; document.body.style.pointerEvents='none'; }
function unlock(){ uiLocked=false; document.body.style.pointerEvents='auto'; }

function onCellClick(i){
  if(gameOver) return;
  if(teachingMode){
    const mv=SCRIPT[stepIndex]; if(!mv||mv.actor!=='blue') return;
    if(uiLocked) return;
    clearHints(); clearArrow();
    if(mv.type==='place'){
      const dst=boardEl.children[mv.to]; dst&&dst.classList.add("hint");
      lock();
      setTimeout(()=>{
        const trayBtn=document.querySelector(`#trayBlue .tray-btn[data-size="${mv.size}"]`);
        const dot=trayBtn?trayBtn.querySelector('.mini'):trayBtn;
        const dstEl=boardEl.children[mv.to];
        ghostMove(dot,dstEl,'blue',mv.size,600).then(()=>{
          board[mv.to].push({player:'blue',size:mv.size});
          counts.blue[mv.size]--;
          audio.sfxPlace('blue', mv.size);
          stepIndex++; clearHints(); clearTrayGlow(); clearArrow();
          if(checkWin('blue')){ startWinSequence(); unlock(); return; }
          current='orange'; render(); setTimeout(runAIMoveIfAny,450);
        });
      },CONFIRM_DELAY);
    }else{
      const src=boardEl.children[mv.from],dst=boardEl.children[mv.to];
      src&&src.classList.add("source-cue"); dst&&dst.classList.add("hint-move");
      const pos=getCenter(src); lock();
      setTimeout(()=>{
        board[mv.from].pop(); render();
        ghostMove({x:pos.x,y:pos.y},dst,'blue',mv.size,650).then(()=>{
          board[mv.to].push({player:'blue',size:mv.size});
          audio.sfxMove('blue', mv.size);
          stepIndex++; movingFromIndex=null; clearHints(); clearArrow();
          if(checkWin('blue')){ startWinSequence(); unlock(); return; }
          current='orange'; render(); setTimeout(runAIMoveIfAny,450);
        });
      },CONFIRM_DELAY);
    }
    return;
  }
  handlePVP(i);
}

function runAIMoveIfAny(){
  if(gameOver||stepIndex>=SCRIPT.length){showNextHint();unlock();return}
  const mv=SCRIPT[stepIndex]; if(mv.actor!=='orange'){showNextHint();unlock();return}
  showNextHint(true);
  if(mv.type==='place'){
    const trayBtn=[...document.querySelectorAll('#trayOrange .tray-btn')].find(b=>Number(b.dataset.size)===mv.size);
    const dot=trayBtn?trayBtn.querySelector('.mini'):trayBtn;
    const dst=boardEl.children[mv.to];
    ghostMove(dot,dst,'orange',mv.size,600).then(()=>{
      board[mv.to].push({player:'orange',size:mv.size});
      counts.orange[mv.size]--;
      audio.sfxPlace('orange', mv.size);
      stepIndex++; clearTrayGlow(); clearHints(); clearArrow();
      if(checkWin('orange')){ gameOver=true; render(); alert("橙方勝"); unlock(); return; }
      current='blue'; render(); showNextHint(); unlock();
    });
  }else{
    const src=boardEl.children[mv.from],dst=boardEl.children[mv.to],pos=getCenter(src);
    board[mv.from].pop(); render();
    ghostMove({x:pos.x,y:pos.y},dst,'orange',mv.size,650).then(()=>{
      board[mv.to].push({player:'orange',size:mv.size});
      audio.sfxMove('orange', mv.size);
      stepIndex++; movingFromIndex=null; clearHints(); clearArrow();
      if(checkWin('orange')){ gameOver=true; render(); alert("橙方勝"); unlock(); return; }
      current='blue'; render(); showNextHint(); unlock();
    });
  }
}

function startWinSequence(){
  gameOver=true; clearArrow(); clearHints(); clearTrayGlow();
  winLineIdx=getWinningLine('blue'); if(!winLineIdx) return;
  document.body.classList.add('win-spotlight');
  winPulse=new Set(winLineIdx); render();
  audio.sfxWin();
  setTimeout(()=>{ winPulse.clear(); render(); toYCHAndBanners(); }, WIN_DELAY);
}

function toYCHAndBanners(){
  document.body.classList.remove('win-spotlight');
  const winSet=new Set(winLineIdx);
  for(let i=0;i<9;i++){
    if(!winSet.has(i) && board[i].length){
      const t=topPiece(i);
      if(t && t.player==='orange'){
        board[i].pop();
        const t2=topPiece(i);
        if(t2 && t2.player==='blue') board[i].pop();
      }
    }
  }
  render();
  const pts=winLineIdx.map(i=>({i,...getCenter(boardEl.children[i])}));
  pts.sort((a,b)=>a.x!==b.x?(a.x-b.x):(a.y-b.y));
  const letters=["Y","C","H"];
  pts.forEach((p,idx)=>{
    board[p.i]=[]; winLetters[p.i]=letters[idx];
    const cell=boardEl.children[p.i],content=cell.querySelector('.cell-content'),overlay=cell.querySelector('.cell-overlay');
    content.innerHTML=""; overlay.innerHTML="";
    const s=document.createElement('span'); s.className='win-letter'; s.textContent=letters[idx];
    s.style.animationDelay=(idx*180)+'ms'; overlay.appendChild(s);
    setTimeout(()=>{ overlay.innerHTML=""; const st=document.createElement('span'); st.className='win-letter-still'; st.textContent=letters[idx]; overlay.appendChild(st); }, 700+idx*180);
  });
}

function handlePVP(index){
  if(gameOver) return;
  const tp=topPiece(index);
  if(selectedSize!==null){
    if(!canPlace(current,selectedSize,index)){ hint("不能覆蓋同等或更大"); return; }
    board[index].push({player:current,size:selectedSize}); counts[current][selectedSize]--;
    selectedSize=null; clearTrayGlow(); render();
    if(checkWin(current)){ alert((current==='blue'?'綠':'橙')+'方勝'); gameOver=true; return; }
    switchTurn(); return;
  }
  if(pvpSelectedFrom===null){
    if(!tp){ hint("請先選托盤落子，或揀你棋子再移動"); return; }
    if(tp.player!==current){ hint("唔到你移對家棋"); return; }
    pvpSelectedFrom=index; boardEl.children[index].classList.add('source-cue'); return;
  }
  if(index===pvpSelectedFrom){
    boardEl.children[index].classList.remove('source-cue'); pvpSelectedFrom=null; hint("已取消選擇"); return;
  }
  const fromTop=topPiece(pvpSelectedFrom);
  if(!fromTop){ pvpSelectedFrom=null; clearHints(); return; }
  if(!canMove(current,fromTop.size,pvpSelectedFrom,index)){ hint("移動不合法（只能大吃小）"); return; }
  board[index].push(board[pvpSelectedFrom].pop()); clearHints(); pvpSelectedFrom=null; render();
  if(checkWin(current)){ alert((current==='blue'?'綠':'橙')+'方勝'); gameOver=true; return; }
  switchTurn();
}

function hint(t){
  msgEl.textContent=t;
  msgEl.classList.add('show');
  clearTimeout(hint._t);
  hint._t=setTimeout(()=>msgEl.classList.remove('show'),1400);
}

function redrawArrowIfAny(){ if(!currentArrow) return; const {fromEl,toEl,kind}=currentArrow; drawArrow(fromEl,toEl,kind); }
function followGhostDuringAnim(){
  if(!ghostAnim) return;
  const now=Date.now(); if(now>ghostAnim.endTime){ ghostAnim=null; return; }
  const B=getCenter(ghostAnim.toEl); if(B){ ghostAnim.el.style.left=B.x+"px"; ghostAnim.el.style.top=B.y+"px"; }
  requestAnimationFrame(followGhostDuringAnim);
}
function viewportSync(){ layoutArrowLayer(); redrawArrowIfAny(); followGhostDuringAnim(); }
['resize','scroll','orientationchange'].forEach(ev=>{ window.addEventListener(ev, viewportSync, {passive:true}); });
if(window.visualViewport){
  visualViewport.addEventListener('resize', viewportSync, {passive:true});
  visualViewport.addEventListener('scroll', viewportSync, {passive:true});
}
const ro=new ResizeObserver(viewportSync);
ro.observe(document.documentElement); ro.observe(document.body); ro.observe(boardEl);

layoutArrowLayer();
resetTeaching();

/* ✅ 互動解鎖 AudioContext（iOS/Chrome） */
const unlockAudioOnce=()=>{
  audio.ensureCtx();
  if(audio.ctx && audio.ctx.state==='suspended'){ audio.ctx.resume().catch(()=>{}); }
  window.removeEventListener('pointerdown', unlockAudioOnce);
  window.removeEventListener('keydown', unlockAudioOnce);
};
window.addEventListener('pointerdown', unlockAudioOnce, {once:true});
window.addEventListener('keydown', unlockAudioOnce, {once:true});

})();
</script>
</body>
</html>
