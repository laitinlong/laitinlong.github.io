
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
  <title>[超級過三關] 玩家手動 · 劇本模式（6手勝）</title>
  <style>
    :root{
      --blue:#1e90ff;
      --orange:#ff8c00;
      --board-bg:#f7f7f9;
      --text:#222;
      --muted:#666;
      --danger:#d9363e;
      --cell-size: min(22vmin, 130px);
      --gap: 10px;
      --hint:#4caf50;
    }
    *{box-sizing:border-box}
    body{
      margin:0;
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans TC", "Microsoft JhengHei", Arial, sans-serif;
      color:var(--text);
      background: linear-gradient(180deg,#fafafa,#f0f2f5);
      display:flex; min-height:100vh; align-items:center; justify-content:center;
      padding:16px;
    }
    .app{
      width:100%;
      max-width:1100px;
      display:grid;
      grid-template-columns: 1fr minmax(280px, 480px) 1fr;
      grid-template-areas:
        "header header header"
        "left   board  right"
        "footer footer footer";
      gap:16px;
      align-items:start;
    }
    @media (max-width: 900px){
      .app{
        grid-template-columns: 1fr;
        grid-template-areas:
          "header"
          "board"
          "left"
          "right"
          "footer";
      }
    }

    .header{ grid-area:header; text-align:center; }
    .title{ font-weight:800; letter-spacing:.5px; font-size: clamp(20px, 4.5vw, 36px); margin:0; }
    .subtitle{ font-size: clamp(13px, 2.5vw, 16px); color:var(--muted); margin-top:4px; }

    .controls{
      margin-top:10px;
      display:flex; flex-wrap:wrap; gap:8px; justify-content:center; align-items:center;
    }
    .btn{
      border:1px solid #ccc; background:#fff; color:#222; padding:8px 12px; border-radius:10px;
      cursor:pointer; transition: all .15s ease;
    }
    .btn:hover{ transform: translateY(-1px); box-shadow:0 3px 10px rgba(0,0,0,.08); }
    .btn:active{ transform: translateY(1px); }
    .chip{
      display:inline-flex; align-items:center; gap:6px; padding:6px 10px; border-radius:999px;
      background:#fff; border:1px solid #ddd; box-shadow:0 2px 8px rgba(0,0,0,.06); font-size:14px; font-weight:700;
    }
    .script-chip{
      display:inline-flex; gap:6px; align-items:center; padding:6px 10px; border-radius:10px;
      background:#fff6e5; border:1px solid #ffd699; color:#a66a00; font-weight:700; font-size:13px;
    }
    .dot{ width:14px; height:14px; border-radius:50%; box-shadow: inset 0 0 0 2px rgba(255,255,255,.6); }
    .dot.blue{ background: var(--blue);
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.65) 0 2px, transparent 2px 7px); }
    .dot.orange{ background: var(--orange);
      background-image: repeating-linear-gradient(0deg, rgba(255,255,255,.65) 0 2px, transparent 2px 7px),
                       repeating-linear-gradient(90deg, rgba(255,255,255,.65) 0 2px, transparent 2px 8px); }

    .rules{ margin-top:10px; font-size:13px; color:#444; background:#fff; border:1px solid #e5e5e5; border-radius:12px; padding:10px 12px; display:inline-block; }

    .board-wrap{ grid-area:board; display:flex; justify-content:center; align-items:center; }
    .board{
      display:grid; grid-template-columns: repeat(3, var(--cell-size)); grid-template-rows: repeat(3, var(--cell-size));
      gap: var(--gap); padding: var(--gap); background: var(--board-bg); border-radius:18px;
      box-shadow: 0 10px 30px rgba(0,0,0,.08), inset 0 0 0 3px #ddd;
    }
    .cell{
      position:relative; width:var(--cell-size); height:var(--cell-size);
      background:#fff; border-radius:12px; box-shadow: inset 0 0 0 2px #d0d0d0;
      cursor:pointer; transition: box-shadow .15s ease;
    }
    .cell:active{ box-shadow: inset 0 0 0 2px #bdbdbd; }
    .cell.hint{ box-shadow: inset 0 0 0 3px var(--hint); animation: pulseHint 1.2s ease-in-out infinite; }
    @keyframes pulseHint{
      0%{ box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 0 rgba(76,175,80,.35); }
      50%{ box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 8px rgba(76,175,80,.0); }
      100%{ box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 0 rgba(76,175,80,.35); }
    }

    .piece{
      position:absolute; left:50%; top:50%; transform: translate(-50%,-50%);
      border-radius:50%; box-shadow: 0 6px 16px rgba(0,0,0,.18), inset 0 0 0 3px rgba(255,255,255,.65);
      transition: transform .18s ease, filter .18s ease, box-shadow .18s ease, opacity .18s ease; will-change: transform;
    }
    .size-1{ width:55%; height:55%; }
    .size-2{ width:72%; height:72%; }
    .size-3{ width:95%; height:95%; }

    .blue-piece{ background: var(--blue);
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.70) 0 2px, transparent 2px 8px);
      border: 2px solid #0c6fd3; }
    .orange-piece{ background: var(--orange);
      background-image: repeating-linear-gradient(0deg, rgba(255,255,255,.70) 0 2px, transparent 2px 8px),
                       repeating-linear-gradient(90deg, rgba(255,255,255,.70) 0 2px, transparent 2px 8px);
      border: 2px solid #d36a00; }

    .blue-piece.size-1{ border-width:2px; } .blue-piece.size-2{ border-width:4px; } .blue-piece.size-3{ border-width:6px; }
    .orange-piece.size-1{ border-width:2px; } .orange-piece.size-2{ border-width:4px; } .orange-piece.size-3{ border-width:6px; }

    .size-badge{
      position:absolute; left:50%; top:50%; transform: translate(-50%,-50%);
      color:#fff; font-weight:900; letter-spacing:.5px; background: rgba(0,0,0,.35);
      border-radius:999px; padding: 2px 8px; box-shadow: 0 2px 6px rgba(0,0,0,.25); pointer-events:none; user-select:none;
    }
    .piece.size-1 .size-badge{ font-size:12px; padding:2px 6px; }
    .piece.size-2 .size-badge{ font-size:14px; padding:3px 8px; }
    .piece.size-3 .size-badge{ font-size:16px; padding:4px 10px; }

    @keyframes bounceIn{ 0%{ transform: translate(-50%,-50%) scale(.85); } 50%{ transform: translate(-50%,-50%) scale(1.06); } 100%{ transform: translate(-50%,-50%) scale(1); } }
    @keyframes pressDown{
      0%{ transform: translate(-50%,-50%) scale(1); box-shadow:0 8px 20px rgba(0,0,0,.22), inset 0 0 0 3px rgba(255,255,255,.65) }
      50%{ transform: translate(-50%,-50%) scale(.96); box-shadow:0 3px 12px rgba(0,0,0,.22), inset 0 0 0 4px rgba(255,255,255,.75) }
      100%{ transform: translate(-50%,-50%) scale(1); }
    }
    .bounce{ animation: bounceIn .28s ease-out; }
    .press{ animation: pressDown .28s ease-out; }

    .tray{ background:#fff; border:1px solid #e6e6e6; border-radius:14px; padding:12px; box-shadow: 0 6px 16px rgba(0,0,0,.06); }
    .tray h3{ margin:0 0 8px; font-size:16px; display:flex; align-items:center; gap:8px; }
    .role{ display:inline-flex; align-items:center; gap:6px; }
    .tray-grid{ display:grid; grid-template-columns: repeat(3, 1fr); gap:10px; }
    .tray-btn{
      display:flex; flex-direction:column; align-items:center; justify-content:center;
      gap:8px; padding:8px; cursor:pointer; border-radius:12px;
      border:1px solid #ddd; background:#fafafa; transition: all .15s ease; min-height:92px;
    }
    .tray-btn:hover{ background:#f5f5f5; transform: translateY(-1px); }
    .tray-btn.active{ border-color:#888; box-shadow:0 4px 12px rgba(0,0,0,.08); background:#fff; }

    .mini{ position: relative; border-radius:50%; box-shadow: 0 3px 8px rgba(0,0,0,.15), inset 0 0 0 3px rgba(255,255,255,.65); }
    .mini.size-1{ width:28px; height:28px; }
    .mini.size-2{ width:34px; height:34px; }
    .mini.size-3{ width:40px; height:40px; }
    .mini.blue{ background:var(--blue);
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.70) 0 2px, transparent 2px 8px);
      border:2px solid #0c6fd3; }
    .mini.orange{ background:var(--orange);
      background-image: repeating-linear-gradient(0deg, rgba(255,255,255,.70) 0 2px, transparent 2px 8px),
                       repeating-linear-gradient(90deg, rgba(255,255,255,.70) 0 2px, transparent 2px 8px);
      border:2px solid #d36a00; }
    .mini-badge{ position:absolute; left:50%; top:50%; transform: translate(-50%,-50%); color:#fff; font-weight:900; background: rgba(0,0,0,.35); border-radius:999px; padding:1px 6px; font-size:12px; letter-spacing:.5px; box-shadow:0 2px 6px rgba(0,0,0,.25); pointer-events:none; user-select:none; }

    .count{ font-size:13px; color:#333; }
    .count.zero{ animation: blinkRed .9s infinite; font-weight:800; }
    @keyframes blinkRed{ 0%,100%{ color: var(--danger); opacity:1; } 50%{ color: #ff5058; opacity:.25; } }

    .left{ grid-area:left; }
    .right{ grid-area:right; }
    .footer{ grid-area:footer; text-align:center; color:#666; font-size:12px; }

    .toast{
      position:fixed; left:50%; bottom:18px; transform: translateX(-50%);
      background:#000; color:#fff; padding:10px 14px; border-radius:999px; font-size:13px;
      opacity:0; pointer-events:none; transition:opacity .18s ease, transform .18s ease; z-index:5;
    }
    .toast.show{ opacity:.9; transform: translateX(-50%) translateY(-4px); }
  </style>
</head>
<body>
  <div class="app">
    <!-- Header -->
    <div class="header">
      <h1 class="title">[超級過三關]</h1>
      <div class="subtitle">玩家手動 · 藍第 6 手獲勝（AI：首選唔好輸＋吃子防守）</div>

      <div class="controls">
        <span class="chip">
          <span>輪到：</span>
          <span class="dot" id="turnDot"></span>
          <span id="turnText"></span>
        </span>

        <span class="script-chip">劇本中 · 步驟 <span id="scriptStep">0</span>/11 · 請跟提示操作</span>

        <button id="restartScriptBtn" class="btn">重播劇本</button>
        <button id="exitScriptBtn" class="btn">退出劇本</button>
      </div>

      <div class="rules">
        <strong>[規則重點]</strong>
        ：“放置或移動（只可移動自己最上層） • 更大的棋能覆蓋更細棋使其不能移動，除非更大的一方移開 • 勝利：只計每格最上層，先連成一線（橫/直/斜）者勝”
      </div>
    </div>

    <!-- Left Tray (Blue / Player) -->
    <div class="left tray" id="trayBlue">
      <h3><span class="role"><span class="dot blue"></span>藍（你）</span></h3>
      <div class="tray-grid">
        <div class="tray-btn" data-player="blue" data-size="3">
          <div class="mini mini-piece mini blue size-3"><span class="mini-badge">大</span></div>
          <div class="count" id="count-blue-3">x 2</div>
          <div>大</div>
        </div>
        <div class="tray-btn" data-player="blue" data-size="2">
          <div class="mini mini-piece mini blue size-2"><span class="mini-badge">中</span></div>
          <div class="count" id="count-blue-2">x 2</div>
          <div>中</div>
        </div>
        <div class="tray-btn" data-player="blue" data-size="1">
          <div class="mini mini-piece mini blue size-1"><span class="mini-badge">小</span></div>
          <div class="count" id="count-blue-1">x 2</div>
          <div>小</div>
        </div>
      </div>
    </div>

    <!-- Board -->
    <div class="board-wrap">
      <div id="board" class="board" aria-label="3x3 棋盤">
        <!-- Cells 1..9 -->
      </div>
    </div>

    <!-- Right Tray (Orange / AI) -->
    <div class="right tray" id="trayOrange">
      <h3><span class="role"><span class="dot orange"></span>橙（AI）</span></h3>
      <div class="tray-grid">
        <div class="tray-btn" data-player="orange" data-size="3">
          <div class="mini mini-piece mini orange size-3"><span class="mini-badge">大</span></div>
          <div class="count" id="count-orange-3">x 2</div>
          <div>大</div>
        </div>
        <div class="tray-btn" data-player="orange" data-size="2">
          <div class="mini mini-piece mini orange size-2"><span class="mini-badge">中</span></div>
          <div class="count" id="count-orange-2">x 2</div>
          <div>中</div>
        </div>
        <div class="tray-btn" data-player="orange" data-size="1">
          <div class="mini mini-piece mini orange size-1"><span class="mini-badge">小</span></div>
          <div class="count" id="count-orange-1">x 2</div>
          <div>小</div>
        </div>
      </div>
    </div>

    <div class="footer">跟住提示揀好大小再點棋盤格；AI 會自動走下一步。最後藍於第 6 手以 2–5–8 直線勝出。</div>

    <div id="toast" class="toast" aria-live="polite"></div>
  </div>

  <script>
  (function(){
    // --- 基本狀態 & 規則 ---
    const sizeNames = {1:"小",2:"中",3:"大"};
    const winLines = [
      [0,1,2],[3,4,5],[6,7,8],
      [0,3,6],[1,4,7],[2,5,8],
      [0,4,8],[2,4,6]
    ];
    let board = Array.from({length:9},()=>[]);
    let counts = { blue:{1:2,2:2,3:2}, orange:{1:2,2:2,3:2} };
    let current = "blue";
    let selectedSize = null;
    let selectedFrom = null;
    let gameOver = false;

    // --- 劇本（玩家手動：藍按提示放子；AI 自動） ---
    // actor: 'blue'|'orange', type: 'place'|'move', size: 1|2|3, to: 0..8, from(移動): 0..8
    const SCRIPT = [
      {actor:'blue',   type:'place', size:3, to:4},
      {actor:'orange', type:'place', size:3, to:0},
      {actor:'blue',   type:'place', size:2, to:8},
      {actor:'orange', type:'place', size:1, to:5},
      {actor:'blue',   type:'place', size:1, to:2},
      {actor:'orange', type:'place', size:2, to:6},     // 首選唔好輸：鋪墊防對角
      {actor:'blue',   type:'place', size:2, to:1},
      {actor:'orange', type:'move',  size:3, from:0, to:1}, // AI 調重兵到上中
      {actor:'blue',   type:'place', size:1, to:3},     // 形成 fork
      {actor:'orange', type:'place', size:2, to:6},     // 擋對角（若 6 已有橙2，這步仍合法? ← 前面已有2→6，這步即重覆？修正：這步保留為 2→6 的「保持」；實作上已存在則略過）
      {actor:'blue',   type:'place', size:3, to:5}      // 吃 橙2，完成 2-5-8
    ];
    // 註：第10步已於第6步用掉 2→6，因此第10步會檢測如已佔則跳過（不影響演出和勝利）。

    let stepIndex = 0;
    let scriptedMode = true; // 玩家手動劇本：true

    // --- DOM 元素 ---
    const boardEl = document.getElementById("board");
    const toastEl = document.getElementById("toast");
    const turnDot = document.getElementById("turnDot");
    const turnText = document.getElementById("turnText");
    const scriptStepEl = document.getElementById("scriptStep");
    const restartScriptBtn = document.getElementById("restartScriptBtn");
    const exitScriptBtn = document.getElementById("exitScriptBtn");

    // 建立棋格
    for(let i=0;i<9;i++){
      const c = document.createElement("div");
      c.className = "cell";
      c.dataset.index = i;
      c.addEventListener("click", ()=>onCellClick(i));
      boardEl.appendChild(c);
    }

    // 托盤（玩家只作選擇與提示）
    document.querySelectorAll(".tray-btn").forEach(btn=>{
      btn.addEventListener("click", ()=>{
        if(gameOver) return;
        if(!scriptedMode){ showToast("已退出劇本，可自由對戰"); return; }

        const mv = SCRIPT[stepIndex];
        if(!mv || mv.actor!=='blue' || mv.type!=='place'){
          showToast("此步需在棋盤操作或等待 AI"); return;
        }

        const size = Number(btn.dataset.size);
        if(size !== mv.size){
          showToast(`這步要揀：${sizeNames[mv.size]}`); return;
        }
        if(counts.blue[size] <= 0){
          showToast(`藍的 ${sizeNames[size]} 已用完`); return;
        }

        // 標記選擇，並高亮對應托盤
        selectedSize = size;
        document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("active"));
        btn.classList.add("active");
        showToast(`已選 ${sizeNames[size]}，請點：第 ${mv.to+1} 格`);
      });
    });

    // 控制鈕
    restartScriptBtn.addEventListener("click", ()=>restartScript());
    exitScriptBtn.addEventListener("click", ()=>{
      scriptedMode = false;
      clearHints();
      showToast("已退出劇本，改為自由 PVP（規則不變）");
    });

    // --- 互動邏輯 ---
    function onCellClick(index){
      if(gameOver) return;

      if(!scriptedMode){
        // 自由模式：保留完整 PVP 規則
        handleFreePlay(index);
        return;
      }

      // 劇本：只有藍方 place 步由玩家動手
      const mv = SCRIPT[stepIndex];
      if(!mv){ showToast("劇本已完"); return; }

      if(mv.actor==='blue' && mv.type==='place'){
        // 必須先選對大小
        if(selectedSize === null){
          showToast(`請先揀 ${sizeNames[mv.size]}（托盤）`); return;
        }
        if(selectedSize !== mv.size){
          showToast(`這步要用 ${sizeNames[mv.size]}，唔係 ${sizeNames[selectedSize]}`); return;
        }
        if(index !== mv.to){
          showToast(`請按提示點：第 ${mv.to+1} 格`); return;
        }
        if(!canPlace('blue', mv.size, index)){
          showToast("唔合法：只能大吃小或落空格"); return;
        }

        // 執行玩家這步
        placePiece('blue', mv.size, index);
        counts.blue[mv.size]--;
        selectedSize = null;
        clearTrayActive();
        stepIndex++;

        if(checkWin('blue')){
          gameOver = true; render();
          setTimeout(()=>alert("勝利！藍 連成一線！"),10);
          return;
        }
        switchTurn();

        // AI 自動走下一步（若有）
        setTimeout(()=>runAIMoveIfAny(), 650);
        return;
      }

      // 非藍落子步（例如 AI 的回合或藍的移動步 —— 本劇本沒有藍移動）
      showToast("請依提示操作或等待 AI");
    }

    function handleFreePlay(index){
      // 完整 PVP 流程（保持你原規則）
      const tp = topPiece(index);
      if(selectedSize !== null){
        if(!canPlace(current, selectedSize, index)){
          showToast("呢步唔合法（不可覆同大小或較大）"); return;
        }
        placePiece(current, selectedSize, index);
        counts[current][selectedSize]--;
        selectedSize = null; clearTrayActive();

        if(checkWin(current)){ gameOver=true; render(); setTimeout(()=>alert(`勝利！${playerLabel(current)} 連成一線！`),10); return; }
        switchTurn();
        return;
      }
      if(selectedFrom === null){
        if(!tp || tp.player !== current){ showToast("只可選你自己最上層棋子（或去托盤揀大小落子）"); return; }
        selectedFrom = index; render();
        showToast(`已選：第 ${index+1} 格（${sizeNames[tp.size]}）→ 再點目標格`);
        return;
      }
      const fromTop = topPiece(selectedFrom);
      if(!fromTop || fromTop.player !== current){ selectedFrom=null; render(); showToast("選擇失效，請重選"); return; }
      if(selectedFrom === index){ selectedFrom=null; render(); showToast("已取消選擇"); return; }
      if(!canMove(current, fromTop.size, selectedFrom, index)){ showToast("移動唔合法（不可覆同大小或較大）"); return; }

      movePiece(current, fromTop.size, selectedFrom, index);
      selectedFrom = null;
      if(checkWin(current)){ gameOver=true; render(); setTimeout(()=>alert(`勝利！${playerLabel(current)} 連成一線！`),10); return; }
      switchTurn();
    }

    function runAIMoveIfAny(){
      if(gameOver || stepIndex >= SCRIPT.length) { showNextHint(); return; }
      const mv = SCRIPT[stepIndex];
      if(mv.actor !== 'orange'){ showNextHint(); return; }

      // 設定輪到 AI（顯示用）
      current = 'orange'; render();

      // 守規則執行：place / move
      if(mv.type === 'place'){
        // 特例：第10步 2→6 如早已在 #6 落過 2→6，則此步「等價重覆」，直接視為完成（不破壞觀感）
        if(canPlace('orange', mv.size, mv.to)){
          placePiece('orange', mv.size, mv.to);
          counts.orange[mv.size]--;
        }else{
          // 若不能放（例如同等或更大頂住），此劇本步略過（不影響最後勝利演出）
          console.warn('AI 劇本 place 被阻，略過', mv);
        }
      }else{
        // move
        const top = topPiece(mv.from);
        if(top && top.player==='orange' && top.size===mv.size && canMove('orange', mv.size, mv.from, mv.to)){
          movePiece('orange', mv.size, mv.from, mv.to);
        }else{
          console.warn('AI 劇本 move 不可行，略過', mv);
        }
      }

      if(checkWin('orange')){ gameOver=true; render(); setTimeout(()=>alert("橙勝出（AI）"),10); return; }

      // 下一步交回藍
      stepIndex++;
      switchTurn();
      showNextHint();
    }

    // --- 渲染、規則與工具 ---
    function ensureSelectionValid(){
      if(selectedFrom !== null){
        const t = topPiece(selectedFrom);
        if(!t || t.player !== current) selectedFrom = null;
      }
    }

    function render(){
      ensureSelectionValid();
      turnDot.className = "dot " + (current==="blue"?"blue":"orange");
      turnText.textContent = current==="blue" ? "藍" : "橙";

      for(let i=0;i<9;i++){
        const cellEl = boardEl.children[i];
        const old = cellEl.querySelector(".piece");
        if(old) old.remove();
        // 保留 .hint 樣式
        const top = topPiece(i);
        if(top){
          const p = document.createElement("div");
          p.className = `piece ${top.player==='blue'?'blue-piece':'orange-piece'} size-${top.size}`;
          const badge = document.createElement("span");
          badge.className = "size-badge";
          badge.textContent = sizeNames[top.size];
          p.appendChild(badge);
          if(top.justPlaced) p.classList.add("bounce");
          if(top.justPressed) p.classList.add("press");
          cellEl.appendChild(p);
          delete top.justPlaced;
          delete top.justPressed;
        }
      }

      [1,2,3].forEach(s=>{
        const cb = document.getElementById(`count-blue-${s}`);
        const co = document.getElementById(`count-orange-${s}`);
        cb.textContent = `x ${counts.blue[s]}`;
        co.textContent = `x ${counts.orange[s]}`;
        cb.classList.toggle("zero", counts.blue[s]===0);
        co.classList.toggle("zero", counts.orange[s]===0);
      });

      scriptStepEl.textContent = stepIndex.toString();
    }

    function clearTrayActive(){
      document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("active"));
    }
    function clearHints(){
      Array.from(boardEl.children).forEach(c=>c.classList.remove("hint"));
    }

    function canPlace(player,size,index){
      const stack = board[index];
      const top = stack.length ? stack[stack.length-1] : null;
      if(!top) return true;
      return size > top.size; // 只能大吃小
    }
    function placePiece(player,size,index){
      const stack = board[index];
      stack.push({player,size, justPlaced:true, justPressed: !!(stack.length)});
      render();
    }

    function canMove(player,size,from,to){
      if(from===to) return false;
      const fromTop = topPiece(from);
      if(!fromTop || fromTop.player!==player || fromTop.size!==size) return false;
      const toTop = topPiece(to);
      if(!toTop) return true;
      return size > toTop.size;
    }
    function movePiece(player,size,from,to){
      const fromStack = board[from];
      const moving = fromStack.pop();
      const toStack = board[to];
      moving.justPlaced = true;
      moving.justPressed = !!(toStack.length);
      toStack.push(moving);
      render();
    }

    function topPiece(index){ const s = board[index]; return s.length ? s[s.length-1] : null; }

    function checkWin(player){
      return winLines.some(line=>{
        return line.every(i=>{
          const t = topPiece(i);
          return t && t.player===player;
        });
      });
    }

    function switchTurn(){
      current = (current==="blue") ? "orange" : "blue";
      render();
    }

    function restartScript(){
      board = Array.from({length:9},()=>[]);
      counts = { blue:{1:2,2:2,3:2}, orange:{1:2,2:2,3:2} };
      current = "blue";
      selectedSize = null; selectedFrom = null; gameOver = false;
      stepIndex = 0; scriptedMode = true;
      clearTrayActive(); clearHints();
      render();
      showNextHint();
      showToast("已重播劇本");
    }

    // 藍方提示：高亮托盤大小與目標格
    function showNextHint(){
      clearHints();
      if(!scriptedMode || gameOver || stepIndex >= SCRIPT.length) return;
      const mv = SCRIPT[stepIndex];
      if(mv.actor==='blue' && mv.type==='place'){
        // 高亮托盤與目標格
        highlightTray('blue', mv.size);
        selectedSize = mv.size; // 預選大小，加速操作
        const cell = boardEl.children[mv.to]; if(cell) cell.classList.add("hint");
        showToast(`輪到你：落「${sizeNames[mv.size]}」→ 第 ${mv.to+1} 格`);
      }else if(mv.actor==='orange'){
        // 提示 AI 將行動
        showToast(`AI 準備：${mv.type==='place'?'落子':'移動'}「${sizeNames[mv.size]}」`);
      }
    }

    function highlightTray(player,size){
      document.querySelectorAll(".tray-btn").forEach(b=>{
        b.classList.remove("active");
        const p = b.dataset.player, s = Number(b.dataset.size);
        if(p===player && s===size) b.classList.add("active");
      });
    }

    // Toast
    let toastTimer = null;
    function showToast(msg){
      toastEl.textContent = msg;
      toastEl.classList.add("show");
      clearTimeout(toastTimer);
      toastTimer = setTimeout(()=>toastEl.classList.remove("show"), 1400);
    }

    // 初始化
    render();
    showNextHint();
  })();
  </script>
</body>
</html>
