
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
  <title>超級過三關</title>
  <style>
    :root{
      --blue:#1e90ff;
      --orange:#ff8c00;
      --board-bg:#f7f7f9;
      --text:#222;
      --cell-size: min(22vmin, 130px);
      --gap: 10px;
      --hint:#43a047;
      --move:#43a047;
      --labelShadow: 0 4px 12px rgba(0,0,0,.15);
      --arrowPlace:#43a047;
      --arrowMove:#43a047;
    }
    *{ box-sizing:border-box }
    body{
      margin:0;
      font-family: system-ui,-apple-system,"Segoe UI",Roboto,"Noto Sans TC","Microsoft JhengHei",Arial,sans-serif;
      background: linear-gradient(180deg,#fafafa,#f0f2f5);
      color:var(--text);
      display:flex; min-height:100vh; align-items:center; justify-content:center;
      padding:16px;
    }
    .app{
      width:100%; max-width:1100px;
      display:grid; gap:16px; align-items:start;
      grid-template-columns: 1fr minmax(280px, 480px) 1fr;
      grid-template-areas:
        "header header header"
        "left   board  right";
    }
    @media (max-width: 900px){
      .app{
        grid-template-columns: 1fr;
        grid-template-areas:
          "header"
          "board"
          "left"
          "right";
      }
    }

    .header{ grid-area:header; display:flex; flex-direction:column; align-items:center; gap:8px; }
    .title{
      margin:0; font-weight:900; letter-spacing:.5px;
      color: var(--blue);
      font-size: clamp(22px, 5vw, 40px);
      text-shadow: 0 2px 10px rgba(30,144,255,.15);
    }
    .header-bar{ display:flex; align-items:center; gap:10px; flex-wrap:wrap; }
    .dot{ width:14px; height:14px; border-radius:50%; box-shadow: inset 0 0 0 2px rgba(255,255,255,.6); }
    .dot.blue{ background: var(--blue); }
    .dot.orange{ background: var(--orange); }
    .turn-text{ font-weight:800; font-size:14px; color:#333; }
    .btn{ border:1px solid #ccc; background:#fff; color:#222; padding:8px 12px; border-radius:10px; cursor:pointer; display:inline-flex; align-items:center; justify-content:center; font-size:14px; transition:.15s; user-select:none; }
    .btn:hover{ transform: translateY(-1px); box-shadow:0 3px 10px rgba(0,0,0,.08); }
    .btn:active{ transform: translateY(1px); }

    .tray{
      grid-area:left;
      background:#fff; border:1px solid #e6e6e6; border-radius:14px;
      padding:12px; box-shadow: 0 6px 16px rgba(0,0,0,.06);
    }
    .right{ grid-area:right; }
    .tray-grid{ display:grid; grid-template-columns: repeat(3, 1fr); gap:10px; }
    .tray-btn{
      display:flex; flex-direction:column; align-items:center; justify-content:center;
      gap:8px; padding:8px; cursor:pointer; border-radius:12px;
      border:1px solid #ddd; background:#fafafa; transition:.15s; min-height:92px;
      text-align:center;
    }
    .tray-btn:hover{ background:#f5f5f5; transform: translateY(-1px); }
    .tray-btn.active{ border-color:#888; box-shadow:0 4px 12px rgba(0,0,0,.08); background:#fff; }

    .mini{
      position: relative; border-radius:50%;
      width:40px; height:40px;
      box-shadow: 0 3px 8px rgba(0,0,0,.15), inset 0 0 0 3px rgba(255,255,255,.65);
    }
    .mini.size-1{ width:28px; height:28px; }
    .mini.size-2{ width:34px; height:34px; }
    .mini.size-3{ width:40px; height:40px; }
    .mini.blue{ background:var(--blue); border:2px solid #0c6fd3; }
    .mini.orange{ background:var(--orange); border:2px solid #d36a00; }

    .mini-badge{
      position:absolute; left:50%; top:50%; transform: translate(-50%,-50%);
      color:#fff; font-weight:900; background: rgba(0,0,0,.35);
      border-radius:999px; padding: 1px 6px; font-size:12px; letter-spacing:.5px;
      box-shadow: 0 2px 6px rgba(0,0,0,.25); user-select:none; pointer-events:none;
      z-index:2;
    }
    .count{ font-size:13px; color:#333; }
    .count.zero{ color:#d9363e; font-weight:800; }

    .tray-btn.glow-green .mini{
      box-shadow:
        0 0 0 4px rgba(67,160,71,.85),
        0 0 14px 2px rgba(67,160,71,.45),
        inset 0 0 0 3px rgba(255,255,255,.70);
      animation: movingPulse 1.1s ease-in-out infinite;
    }

    .board-wrap{ grid-area:board; display:flex; justify-content:center; align-items:center; }
    .board{
      position: relative;
      display:grid; grid-template-columns: repeat(3, var(--cell-size)); grid-template-rows: repeat(3, var(--cell-size));
      gap: var(--gap); padding: var(--gap); background: var(--board-bg); border-radius:18px;
      box-shadow: 0 10px 30px rgba(0,0,0,.08), inset 0 0 0 3px #ddd;
    }
    .cell{
      position:relative; width:var(--cell-size); height:var(--cell-size);
      background:#fff; border-radius:12px; box-shadow: inset 0 0 0 2px #d0d0d0;
      cursor:pointer; transition: box-shadow .15s ease, transform .15s ease;
    }
    .cell:active{ box-shadow: inset 0 0 0 2px #bdbdbd; }

    .cell.hint{
      box-shadow: inset 0 0 0 3px var(--hint);
      animation: pulseHint 1.2s ease-in-out infinite;
    }
    .cell.hint::after{
      content: "放置";
      position:absolute; bottom:8px; right:8px;
      background:var(--hint); color:#fff;
      font-size:16px; font-weight:900; letter-spacing:.5px;
      padding:6px 12px; border-radius:999px;
      box-shadow: var(--labelShadow);
    }
    @keyframes pulseHint{
      0%{ box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 0 rgba(67,160,71,.35); }
      50%{ box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 10px rgba(67,160,71,0); }
      100%{ box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 0 rgba(67,160,71,.35); }
    }

    .cell.hint-move{
      box-shadow: inset 0 0 0 3px var(--move), 0 0 0 6px rgba(67,160,71,.18);
      animation: targetPulse 1.05s ease-in-out infinite;
    }
    .cell.hint-move::after{
      content: "移動";
      position:absolute; bottom:8px; right:8px;
      background:var(--move); color:#fff;
      font-size:16px; font-weight:900; letter-spacing:.5px;
      padding:6px 12px; border-radius:999px;
      box-shadow: var(--labelShadow);
    }
    @keyframes targetPulse{ 0%{ transform:scale(1) } 50%{ transform:scale(1.02) } 100%{ transform:scale(1) } }

    .cell.source-cue{
      box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 6px rgba(67,160,71,.18);
    }

    .piece{
      position:absolute; left:50%; top:50%; transform: translate(-50%,-50%);
      border-radius:50%; overflow:hidden;
      box-shadow: 0 6px 16px rgba(0,0,0,.18), inset 0 0 0 3px rgba(255,255,255,.65);
      transition: transform .18s ease, filter .18s ease, box-shadow .18s ease, opacity .18s ease;
    }
    .size-1{ width:55%; height:55%; }
    .size-2{ width:72%; height:72%; }
    .size-3{ width:95%; height:95%; }
    .blue-piece{ background: var(--blue); border: 2px solid #0c6fd3; }
    .orange-piece{ background: var(--orange); border: 2px solid #d36a00; }

    .blue-piece::before{
      content:"";
      position:absolute; left:50%; top:50%; transform: translate(-50%,-50%);
      width: 72%; height: 72%;
      border-radius: 50%;
      box-shadow: 0 0 0 5px rgba(255,255,255,.95), inset 0 0 0 7px #0c6fd3;
      pointer-events:none; z-index:1;
    }

    .orange-piece::before,
    .orange-piece::after{
      content:"";
      position:absolute; left:50%; top:50%;
      width: 76%; height: 12%;
      background: #d36a00;
      border-radius: 8px;
      transform-origin: center;
      pointer-events:none; z-index:1;
      box-shadow: 0 0 0 4px rgba(255,255,255,.95), 0 1px 2px rgba(0,0,0,.2);
    }
    .orange-piece::before{ transform: translate(-50%,-50%) rotate(45deg); }
    .orange-piece::after { transform: translate(-50%,-50%) rotate(-45deg); }

    .moving-piece{
      box-shadow:
        0 0 0 4px rgba(67,160,71,.85),
        0 0 14px 2px rgba(67,160,71,.45),
        inset 0 0 0 3px rgba(255,255,255,.70);
      animation: movingPulse 1.1s ease-in-out infinite;
    }
    @keyframes movingPulse{
      0%  { transform: translate(-50%,-50%) scale(1); }
      50% { transform: translate(-50%,-50%) scale(1.03); }
      100%{ transform: translate(-50%,-50%) scale(1); }
    }

    .size-badge{
      position:absolute;
      left:50%; top:50%; transform: translate(-50%,-50%);
      color:#fff; font-weight:900; letter-spacing:.5px;
      background: rgba(0,0,0,.35);
      border-radius:999px; padding: 2px 8px;
      box-shadow: 0 2px 6px rgba(0,0,0,.25);
      user-select:none; pointer-events:none;
      z-index: 3;
    }
    .piece.size-1 .size-badge{ font-size:12px; padding:2px 6px; }
    .piece.size-2 .size-badge{ font-size:14px; padding:3px 8px; }
    .piece.size-3 .size-badge{ font-size:16px; padding:4px 10px; }

    .arrow-layer{
      position: fixed; left:0; top:0; width:100vw; height:100vh;
      pointer-events:none; z-index: 9999;
    }
    .arrow-path{
      fill:none; stroke-width:2.5;
      stroke-linecap:round; stroke-linejoin:round;
      stroke-dasharray: 5 12;
      opacity:.8;
      animation: dashMove 1.2s linear infinite;
      filter: drop-shadow(0 1px 2px rgba(0,0,0,.15));
    }
    @keyframes dashMove{ to{ stroke-dashoffset: -14; } }

    .ghost{
      position: fixed; left:0; top:0;
      transform: translate(-50%,-50%);
      transition: left 0.55s ease, top 0.55s ease;
      pointer-events:none; z-index: 9000;
    }
  </style>
</head>
<body>
  <svg id="arrowLayer" class="arrow-layer" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
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
      <h1 class="title">超級過三關</h1>
      <div class="header-bar">
        <span id="turnDot" class="dot blue" aria-hidden="true"></span>
        <span id="turnText" class="turn-text">輪到：藍</span>
        <button id="restartBtn" class="btn" aria-label="重新開始" style="display:none;">重新開始</button>
        <button id="swapBtn" class="btn" aria-label="換邊起手" style="display:none;">換邊起手</button>
        <button id="modeBtn" class="btn" aria-label="切換模式">退出教學模式</button>
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
      <div id="board" class="board" aria-label="3x3 棋盤"></div>
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

  <script>
  (function(){
    const sizeNames = {1:"小",2:"中",3:"大"};
    const winLines = [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];
    const boardEl = document.getElementById("board");
    const turnDot = document.getElementById("turnDot");
    const turnText = document.getElementById("turnText");
    const restartBtn = document.getElementById("restartBtn");
    const swapBtn = document.getElementById("swapBtn");
    const modeBtn = document.getElementById("modeBtn");
    const arrowLayer = document.getElementById('arrowLayer');
    const arrowPath  = document.getElementById('arrowPath');

    let board, counts, current, selectedSize, gameOver;
    let teachingMode = true;
    let stepIndex = 0;
    let movingFromIndex = null;
    let pvpSelectedFrom = null;

    const SCRIPT = [
      {actor:'blue',   type:'place', size:3, to:4},
      {actor:'orange', type:'place', size:3, to:8},
      {actor:'blue',   type:'place', size:3, to:2},
      {actor:'orange', type:'place', size:2, to:6},
      {actor:'blue',   type:'move',  size:3, from:2, to:6},
      {actor:'orange', type:'place', size:2, to:2},
      {actor:'blue',   type:'move',  size:3, from:4, to:2},
      {actor:'orange', type:'place', size:3, to:4},
      {actor:'blue',   type:'place', size:1, to:0},
      {actor:'orange', type:'move',  size:3, from:8, to:0},
      {actor:'blue',   type:'place', size:2, to:8},
      {actor:'orange', type:'move',  size:3, from:4, to:8},
      {actor:'blue',   type:'place', size:2, to:4}
    ];

    function makeCells(){
      if(boardEl.children.length) return;
      for(let i=0;i<9;i++){
        const c = document.createElement("div");
        c.className = "cell";
        c.dataset.index = i;
        c.addEventListener("click", ()=>onCellClick(i));
        boardEl.appendChild(c);
      }
    }

    function resetCommonState(){
      board = Array.from({length:9},()=>[]);
      counts = { blue:{1:2,2:2,3:2}, orange:{1:2,2:2,3:2} };
      current = "blue";
      selectedSize = null;
      gameOver = false;
      movingFromIndex = null;
      pvpSelectedFrom = null;
      render();
      clearHints(); clearArrow(); clearTrayGlow();
    }

    function resetTeaching(){
      teachingMode = true;
      stepIndex = 0;
      modeBtn.textContent = "退出教學模式";
      restartBtn.style.display = "none";
      swapBtn.style.display = "none";
      resetCommonState();
      showNextHint();
    }

    function resetPVP(){
      teachingMode = false;
      modeBtn.textContent = "開始教學模式";
      restartBtn.style.display = "";
      swapBtn.style.display = "";
      resetCommonState();
    }

    restartBtn.addEventListener("click", ()=>{ if(teachingMode) return; resetPVP(); });
    swapBtn.addEventListener("click", ()=>{ if(teachingMode) return; current = (current==="blue")?"orange":"blue"; render(); });
    modeBtn.addEventListener("click", ()=>{ if(teachingMode) resetPVP(); else resetTeaching(); });

    document.querySelectorAll(".tray-btn").forEach(btn=>{
      btn.addEventListener("click", ()=>{
        if(gameOver) return;
        const player = btn.dataset.player;
        const size = Number(btn.dataset.size);
        if(teachingMode){
          const mv = SCRIPT[stepIndex];
          if(!mv || mv.actor!=='blue' || mv.type!=='place') return;
          if(size !== mv.size) return;
          if(counts.blue[size] <= 0) return;
          selectedSize = size;
          showNextHint();
        }else{
          if(player !== current) return;
          if(counts[player][size] <= 0) return;
          selectedSize = size;
          clearTrayGlow();
          btn.classList.add("glow-green","active");
        }
      });
    });

    function getCenter(el){ if(!el) return null; const r=el.getBoundingClientRect(); return {x:r.left+r.width/2,y:r.top+r.height/2,w:r.width,h:r.height}; }
    function setSvgSize(){ arrowLayer.setAttribute('viewBox', `0 0 ${window.innerWidth} ${window.innerHeight}`); }
    function offsetEndpoints(fromEl, toEl){
      const fa=getCenter(fromEl), tb=getCenter(toEl); if(!fa||!tb) return null;
      const dx=tb.x-fa.x, dy=tb.y-fa.y, len=Math.hypot(dx,dy)||1;
      const nx=-dy/len, ny=dx/len; const baseW=Math.min(fa.w||0,tb.w||0)||60; const offset=Math.min(22, baseW*0.10);
      const f2={x:fa.x+nx*offset,y:fa.y+ny*offset}, t2={x:tb.x-nx*offset,y:tb.y-ny*offset};
      return { f:f2, t:t2, mid:{x:(f2.x+t2.x)/2,y:(f2.y+t2.y)/2}, normal:{x:nx,y:ny}, len };
    }
    function drawArrow(fromEl, toEl, kind){
      if(!fromEl||!toEl){ clearArrow(); return; }
      setSvgSize();
      const pts=offsetEndpoints(fromEl,toEl); if(!pts){ clearArrow(); return; }
      const {f,t,mid,normal,len}=pts; const bend=Math.min(28,len*0.10); const cx=mid.x+normal.x*bend, cy=mid.y+normal.y*bend;
      const d=`M ${f.x},${f.y} Q ${cx},${cy} ${t.x},${t.y}`;
      arrowPath.setAttribute('d', d);
      arrowPath.setAttribute('stroke', getComputedStyle(document.documentElement).getPropertyValue(kind==='place'?'--arrowPlace':'--arrowMove') || '#43a047');
      arrowPath.setAttribute('marker-end', `url(#${kind==='place'?'headPlace':'headMove'})`);
      arrowPath.style.opacity='1';
    }
    function clearArrow(){ arrowPath.setAttribute('d',''); arrowPath.style.opacity='0'; }

    window.addEventListener('resize', ()=>{ if(!teachingMode || gameOver) return; showNextHint(true); }, {passive:true});

    function topPiece(i){ const s=board[i]; return s.length?s[s.length-1]:null; }
    function canPlace(player,size,i){ const s=board[i]; const t=s.length?s[s.length-1]:null; if(!t) return true; return size>t.size; }
    function canMove(player,size,from,to){
      if(from===to) return false;
      const ft=topPiece(from); if(!ft||ft.player!==player||ft.size!==size) return false;
      const tt=topPiece(to); if(!tt) return true; return size>tt.size;
    }
    function checkWin(player){ return winLines.some(line=>line.every(i=>{ const t=topPiece(i); return t&&t.player===player; })); }

    function render(){
      turnDot.className = "dot " + (current==="blue"?"blue":"orange");
      turnText.textContent = "輪到：" + (current==="blue"?"藍":"橙");
      for(let i=0;i<9;i++){
        const cellEl=boardEl.children[i];
        cellEl.innerHTML="";
        const top=topPiece(i);
        if(top){
          const p=document.createElement("div");
          p.className=`piece ${top.player==='blue'?'blue-piece':'orange-piece'} size-${top.size}`;
          if(i===movingFromIndex) p.classList.add('moving-piece');
          const badge=document.createElement("span");
          badge.className="size-badge";
          badge.textContent=sizeNames[top.size];
          p.appendChild(badge);
          cellEl.appendChild(p);
        }
      }
      [1,2,3].forEach(s=>{
        const cb=document.getElementById(`count-blue-${s}`);
        const co=document.getElementById(`count-orange-${s}`);
        if(cb){ cb.textContent=`x ${counts.blue[s]}`; cb.classList.toggle("zero", counts.blue[s]===0); }
        if(co){ co.textContent=`x ${counts.orange[s]}`; co.classList.toggle("zero", counts.orange[s]===0); }
      });
    }

    function clearHints(){ Array.from(boardEl.children).forEach(c=>c.classList.remove("hint","hint-move","source-cue")); }
    function clearTrayGlow(){ document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("glow-green","active")); }
    function highlightTray(player,size){
      clearTrayGlow();
      document.querySelectorAll(".tray-btn").forEach(b=>{
        if(b.dataset.player===player && Number(b.dataset.size)===size){ b.classList.add("glow-green","active"); }
      });
    }
    function switchTurn(){ current=(current==="blue")?"orange":"blue"; render(); }

    function ghostMove(fromEl, toEl, player, size, durationMs=550){
      return new Promise((resolve)=>{
        const from=getCenter(fromEl), to=getCenter(toEl); if(!from||!to){ resolve(); return; }
        const cellW = to.w || 80; const ratio=(size===3?0.95: size===2?0.72:0.55); const w=cellW*ratio, h=w;
        const g=document.createElement("div");
        g.className=`piece ${player==='blue'?'blue-piece':'orange-piece'} size-${size} ghost`;
        g.style.left=from.x+"px"; g.style.top=from.y+"px"; g.style.width=w+"px"; g.style.height=h+"px";
        const badge=document.createElement("span"); badge.className="size-badge"; badge.textContent=sizeNames[size]; g.appendChild(badge);
        g.style.transitionDuration = durationMs + "ms";
        document.body.appendChild(g);
        requestAnimationFrame(()=>{ g.style.left=to.x+"px"; g.style.top=to.y+"px"; });
        setTimeout(()=>{ g.remove(); resolve(); }, durationMs+40);
      });
    }

    function showNextHint(keepSelected=false){
      clearHints(); clearArrow(); clearTrayGlow(); movingFromIndex=null;
      if(!teachingMode || gameOver || stepIndex>=SCRIPT.length){ render(); return; }
      const mv=SCRIPT[stepIndex];
      current=mv.actor; render();
      if(mv.type==='place'){
        highlightTray(mv.actor,mv.size);
        const dst=boardEl.children[mv.to]; if(dst) dst.classList.add("hint");
        const trayBtn=[...document.querySelectorAll(`#tray${mv.actor==='blue'?'Blue':'Orange'} .tray-btn`)].find(b=>Number(b.dataset.size)===mv.size);
        const trayDot=trayBtn?trayBtn.querySelector('.mini'):trayBtn;
        drawArrow(trayDot||trayBtn, dst, 'place');
        if(mv.actor==='blue' && !keepSelected) selectedSize=mv.size;
      }else{
        const src=boardEl.children[mv.from]; const dst=boardEl.children[mv.to];
        if(src) src.classList.add("source-cue"); if(dst) dst.classList.add("hint-move");
        movingFromIndex=mv.from; render();
        drawArrow(src, dst, 'move');
      }
    }

    function onCellClick(index){
      if(gameOver) return;
      if(teachingMode){
        const mv=SCRIPT[stepIndex]; if(!mv || mv.actor!=='blue') return;
        if(mv.type==='place'){
          if(selectedSize!==mv.size || index!==mv.to || !canPlace('blue',mv.size,index)) return;
          const trayBtn=[...document.querySelectorAll('#trayBlue .tray-btn')].find(b=>Number(b.dataset.size)===mv.size);
          const trayDot=trayBtn?trayBtn.querySelector('.mini'):trayBtn;
          const dstEl=boardEl.children[mv.to];
          disableUI();
          ghostMove(trayDot,dstEl,'blue',mv.size,550).then(()=>{
            board[mv.to].push({player:'blue',size:mv.size});
            counts.blue[mv.size]--; stepIndex++; clearArrow(); clearTrayGlow(); clearHints();
            if(checkWin('blue')){ gameOver=true; render(); alert("藍方勝"); enableUI(); return; }
            current='orange'; render(); setTimeout(runAIMoveIfAny,450);
          });
        }else{
          if(index!==mv.to || !canMove('blue',mv.size,mv.from,mv.to)) return;
          const srcEl=boardEl.children[mv.from]; const dstEl=boardEl.children[mv.to];
          const fromPos=getCenter(srcEl);
          disableUI();
          board[mv.from].pop(); render(); // 1) 馬上消失
          const tmpSrc=document.createElement('div'); tmpSrc.style.position='fixed'; tmpSrc.style.left=fromPos.x+'px'; tmpSrc.style.top=fromPos.y+'px';
          ghostMove(tmpSrc,dstEl,'blue',mv.size,600).then(()=>{
            board[mv.to].push({player:'blue',size:mv.size});
            stepIndex++; movingFromIndex=null; clearArrow(); clearHints();
            if(checkWin('blue')){ gameOver=true; render(); alert("藍方勝"); enableUI(); return; }
            current='orange'; render(); setTimeout(runAIMoveIfAny,450);
          });
        }
      }else{
        handlePVPClick(index);
      }
    }

    function runAIMoveIfAny(){
      if(gameOver || stepIndex>=SCRIPT.length){ showNextHint(); enableUI(); return; }
      const mv=SCRIPT[stepIndex]; if(mv.actor!=='orange'){ showNextHint(); enableUI(); return; }
      showNextHint(true);
      if(mv.type==='place'){
        const trayBtn=[...document.querySelectorAll('#trayOrange .tray-btn')].find(b=>Number(b.dataset.size)===mv.size);
        const trayDot=trayBtn?trayBtn.querySelector('.mini'):trayBtn;
        const dstEl=boardEl.children[mv.to];
        ghostMove(trayDot,dstEl,'orange',mv.size,550).then(()=>{
          board[mv.to].push({player:'orange',size:mv.size});
          counts.orange[mv.size]--; stepIndex++; clearArrow(); clearTrayGlow(); clearHints();
          if(checkWin('orange')){ gameOver=true; render(); alert("橙方勝"); enableUI(); return; }
          current='blue'; render(); showNextHint(); enableUI();
        });
      }else{
        const srcEl=boardEl.children[mv.from]; const dstEl=boardEl.children[mv.to];
        const fromPos=getCenter(srcEl);
        board[mv.from].pop(); render(); // 1) 馬上消失（AI）
        const tmpSrc=document.createElement('div'); tmpSrc.style.position='fixed'; tmpSrc.style.left=fromPos.x+'px'; tmpSrc.style.top=fromPos.y+'px';
        ghostMove(tmpSrc,dstEl,'orange',mv.size,600).then(()=>{
          board[mv.to].push({player:'orange',size:mv.size});
          stepIndex++; movingFromIndex=null; clearArrow(); clearHints();
          if(checkWin('orange')){ gameOver=true; render(); alert("橙方勝"); enableUI(); return; }
          current='blue'; render(); showNextHint(); enableUI();
        });
      }
    }

    function handlePVPClick(index){
      const tp=topPiece(index);
      if(selectedSize!==null){
        if(!canPlace(current,selectedSize,index)) return;
        board[index].push({player:current,size:selectedSize});
        counts[current][selectedSize]--; selectedSize=null; clearTrayGlow(); render();
        if(checkWin(current)){ alert((current==='blue'?'藍':'橙')+'方勝'); gameOver=true; return; }
        switchTurn(); return;
      }
      if(pvpSelectedFrom===null){
        if(!tp || tp.player!==current) return;
        pvpSelectedFrom=index;
        Array.from(boardEl.children)[index].classList.add('source-cue');
        return;
      }
      if(index===pvpSelectedFrom){ Array.from(boardEl.children)[index].classList.remove('source-cue'); pvpSelectedFrom=null; return; }
      const fromTop=topPiece(pvpSelectedFrom);
      if(!fromTop){ pvpSelectedFrom=null; clearHints(); return; }
      if(!canMove(current,fromTop.size,pvpSelectedFrom,index)) return;
      board[index].push(board[pvpSelectedFrom].pop());
      clearHints(); pvpSelectedFrom=null; render();
      if(checkWin(current)){ alert((current==='blue'?'藍':'橙')+'方勝'); gameOver=true; return; }
      switchTurn();
    }

    let uiLocked=false;
    function disableUI(){ uiLocked=true; document.body.style.pointerEvents='none'; }
    function enableUI(){ uiLocked=false; document.body.style.pointerEvents='auto'; }

    function clearAll(){
      board = Array.from({length:9},()=>[]); render();
    }

    makeCells();
    resetTeaching();
  })();
  </script>
</body>
</html>
