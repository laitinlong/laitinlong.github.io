
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
  <title>[超級過三關] 玩家手動 · 劇本模式（按指定 13 步）</title>
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
    *{ box-sizing:border-box }
    body{
      margin:0;
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans TC", "Microsoft JhengHei", Arial, sans-serif;
      color:var(--text);
      background: linear-gradient(180deg,#fafafa,#f0f2f5);
      display:flex; min-height:100vh; align-items:center; justify-content:center;
      padding:16px;
    }
    .app{
      width:100%; max-width:1100px;
      display:grid; gap:16px; align-items:start;
      grid-template-columns: 1fr minmax(280px, 480px) 1fr;
      grid-template-areas:
        "header header header"
        "left   board  right"
        "footer footer footer";
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

    .controls{ margin-top:10px; display:flex; flex-wrap:wrap; gap:8px; justify-content:center; align-items:center; }
    .btn{ border:1px solid #ccc; background:#fff; color:#222; padding:8px 12px; border-radius:10px; cursor:pointer; transition:.15s; }
    .btn:hover{ transform: translateY(-1px); box-shadow:0 3px 10px rgba(0,0,0,.08); }
    .btn:active{ transform: translateY(1px); }
    .chip{ display:inline-flex; align-items:center; gap:6px; padding:6px 10px; border-radius:999px; background:#fff; border:1px solid #ddd; box-shadow:0 2px 8px rgba(0,0,0,.06); font-size:14px; font-weight:700; }
    .script-chip{ display:inline-flex; gap:6px; align-items:center; padding:6px 10px; border-radius:10px; background:#fff6e5; border:1px solid #ffd699; color:#a66a00; font-weight:700; font-size:13px; }

    .dot{ width:14px; height:14px; border-radius:50%; box-shadow: inset 0 0 0 2px rgba(255,255,255,.6); }
    .dot.blue{ background:var(--blue);
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.65) 0 2px, transparent 2px 7px);}
    .dot.orange{ background:var(--orange);
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
      <div class="subtitle">玩家手動 · 固定劇本（13 步）</div>

      <div class="controls">
        <span class="chip">
          <span>輪到：</span>
          <span class="dot" id="turnDot"></span>
          <span id="turnText"></span>
        </span>

        <span class="script-chip">劇本中 · 步驟 <span id="scriptStep">0</span>/13 · 請跟提示操作</span>

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

    <div class="footer">跟住提示做：有移動（吃子）步要先點來源格再點目標格；AI 會自動跟住劇本落子／移動；第 13 步藍方放中間獲勝。</div>

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

    // --- 你提供的 13 步劇本（玩家手動：藍按提示操作；AI 自動） ---
    // actor: 'blue'|'orange', type: 'place'|'move', size: 1|2|3, to: 0..8, from(移動): 0..8
    const SCRIPT = [
      {actor:'blue',   type:'place', size:3, to:4},        // 1 藍 大→中
      {actor:'orange', type:'place', size:3, to:8},        // 2 橙 大→右下
      {actor:'blue',   type:'place', size:3, to:2},        // 3 藍 大→右上
      {actor:'orange', type:'place', size:2, to:6},        // 4 橙 中→左下
      {actor:'blue',   type:'move',  size:3, from:2, to:6},// 5 藍 移 大 2→6（吃橙中）
      {actor:'orange', type:'place', size:2, to:2},        // 6 橙 中→右上
      {actor:'blue',   type:'move',  size:3, from:4, to:2},// 7 藍 移 大 4→2（吃橙中）
      {actor:'orange', type:'place', size:3, to:4},        // 8 橙 大→中
      {actor:'blue',   type:'place', size:1, to:0},        // 9 藍 小→左上
      {actor:'orange', type:'move',  size:3, from:8, to:0},//10 橙 移 大 8→0（吃藍小）
      {actor:'blue',   type:'place', size:2, to:8},        //11 藍 中→右下
      {actor:'orange', type:'move',  size:3, from:4, to:8},//12 橙 移 大 4→8（吃藍中）
      {actor:'blue',   type:'place', size:2, to:4}         //13 藍 中→中（勝）
    ];
    let stepIndex = 0;
    let scriptedMode = true; // 玩家手動劇本

    // --- DOM ---
    const boardEl = document.getElementById("board");
    const toastEl = document.getElementById("toast");
    const turnDot = document.getElementById("turnDot");
    const turnText = document.getElementById("turnText");
    const scriptStepEl = document.getElementById("scriptStep");
    const restartScriptBtn = document.getElementById("restartScriptBtn");
    const exitScriptBtn = document.getElementById("exitScriptBtn");

    // 建立 9 個格
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
        if(!mv || mv.actor!=='blue'){
          showToast("請按棋盤或等待 AI 行動"); return;
        }
        if(mv.type!=='place'){
          showToast("此步是移動，請先點來源格再點目標格"); return;
        }

        const size = Number(btn.dataset.size);
        if(size !== mv.size){
          showToast(`這步要揀：${sizeNames[mv.size]}`); return;
        }
        if(counts.blue[size] <= 0){
          showToast(`藍的 ${sizeNames[size]} 已用完`); return;
        }

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

    // --- 劇本互動 ---
    function onCellClick(index){
      if(gameOver) return;

      if(!scriptedMode){
        // 自由模式（可照 PVP 規則玩）
        handleFreePlay(index);
        return;
      }

      const mv = SCRIPT[stepIndex];
      if(!mv){ showToast("劇本已完"); return; }

      if(mv.actor==='blue'){
        if(mv.type==='place'){
          // 要先揀對大小
          if(selectedSize === null){ showToast(`請先揀「${sizeNames[mv.size]}」`); return; }
          if(selectedSize !== mv.size){ showToast(`呢步要用「${sizeNames[mv.size]}」`); return; }
          if(index !== mv.to){ showToast(`請點提示格：第 ${mv.to+1} 格`); return; }
          if(!canPlace('blue', mv.size, index)){ showToast("唔合法：只能落空格或大吃小"); return; }

          placePiece('blue', mv.size, index);
          counts.blue[mv.size]--;
          selectedSize = null; clearTrayActive();
          stepIndex++;

          if(checkWin('blue')){ gameOver=true; render(); setTimeout(()=>alert("勝利！藍 連成一線！"),10); return; }
          switchTurn();
          setTimeout(()=>runAIMoveIfAny(), 650);
          return;
        }else{
          // 移動：先點來源，再點目標
          if(selectedFrom===null){
            if(index !== mv.from){ showToast(`請先點來源格：第 ${mv.from+1} 格`); return; }
            const top = topPiece(index);
            if(!top || top.player!=='blue' || top.size!==mv.size){ showToast("來源格頂層唔係你指定大小"); return; }
            selectedFrom = index;
            render();
            showToast(`再點目標格：第 ${mv.to+1} 格`);
            return;
          }else{
            if(index !== mv.to){ showToast(`請點目標格：第 ${mv.to+1} 格`); return; }
            const top = topPiece(selectedFrom);
            if(!top || top.player!=='blue' || top.size!==mv.size){ selectedFrom=null; render(); showToast("來源失效，請重選"); return; }
            if(!canMove('blue', mv.size, selectedFrom, index)){ showToast("移動唔合法（只能大吃小或移去空格）"); return; }

            movePiece('blue', mv.size, selectedFrom, index);
            selectedFrom = null;
            stepIndex++;

            if(checkWin('blue')){ gameOver=true; render(); setTimeout(()=>alert("勝利！藍 連成一線！"),10); return; }
            switchTurn();
            setTimeout(()=>runAIMoveIfAny(), 650);
            return;
          }
        }
      }else{
        showToast("請等待 AI 行動");
      }
    }

    function runAIMoveIfAny(){
      if(gameOver || stepIndex >= SCRIPT.length) { showNextHint(); return; }
      const mv = SCRIPT[stepIndex];
      if(mv.actor !== 'orange'){ showNextHint(); return; }

      // 輪到 AI（顯示用）
      current = 'orange'; render();

      if(mv.type === 'place'){
        if(!canPlace('orange', mv.size, mv.to)){
          console.warn('AI 劇本 place 不合法，終止', mv);
          showToast("（劇本錯誤：AI 放置不合法）");
          return;
        }
        placePiece('orange', mv.size, mv.to);
        counts.orange[mv.size]--;
      }else{
        const top = topPiece(mv.from);
        if(!top || top.player!=='orange' || top.size!==mv.size || !canMove('orange', mv.size, mv.from, mv.to)){
          console.warn('AI 劇本 move 不合法，終止', mv);
          showToast("（劇本錯誤：AI 移動不合法）");
          return;
        }
        movePiece('orange', mv.size, mv.from, mv.to);
      }

      if(checkWin('orange')){ gameOver=true; render(); setTimeout(()=>alert("橙勝出（AI）"),10); return; }
      stepIndex++;
      switchTurn();
      showNextHint();
    }

    // --- 自由 PVP（退出劇本後） ---
    function handleFreePlay(index){
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

    // --- 規則與渲染 ---
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

    // 提示：高亮托盤大小與目標格；移動步高亮來源格（選中）與目標格
    function showNextHint(){
      clearHints();
      if(!scriptedMode || gameOver || stepIndex >= SCRIPT.length) return;
      const mv = SCRIPT[stepIndex];
      if(mv.actor==='blue'){
        if(mv.type==='place'){
          highlightTray('blue', mv.size);
          selectedSize = mv.size; // 預選大小
          const cell = boardEl.children[mv.to]; if(cell) cell.classList.add("hint");
          showToast(`輪到你：落「${sizeNames[mv.size]}」→ 第 ${mv.to+1} 格`);
        }else{
          // 移動：選來源
          selectedFrom = null; // 交互：先點來源
          const src = boardEl.children[mv.from]; if(src) src.classList.add("hint");
          const dst = boardEl.children[mv.to];   if(dst) dst.classList.add("hint");
          showToast(`輪到你：移「${sizeNames[mv.size]}」 第 ${mv.from+1} → 第 ${mv.to+1}`);
        }
      }else{
        showToast(`AI 進行：${mv.type==='place'?'落子':'移動'}「${sizeNames[mv.size]}」`);
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
