<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
  <title>[超級過三關] PVP</title>
  <style>
    :root{
      --blue:#1e90ff;
      --orange:#ff8c00;
      --board-bg:#f7f7f9;
      --line:#222;
      --text:#222;
      --muted:#666;
      --danger:#d9363e;
      --cell-size: min(22vmin, 130px);
      --gap: 10px;
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

    /* Header */
    .header{ grid-area:header; text-align:center; }
    .title{
      font-weight:800; letter-spacing:.5px;
      font-size: clamp(20px, 4.5vw, 36px);
      margin:0;
    }
    .subtitle{
      font-size: clamp(13px, 2.5vw, 16px);
      color:var(--muted);
      margin-top:4px;
    }
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
    .btn.primary{ border-color:#999; font-weight:600; }
    .btn.blue{ color:#fff; background: var(--blue); border-color: var(--blue); }
    .btn.orange{ color:#fff; background: var(--orange); border-color: var(--orange); }

    .status{
      margin-top:8px; font-weight:700;
      display:inline-flex; align-items:center; gap:8px;
    }
    .chip{
      display:inline-flex; align-items:center; gap:6px; padding:6px 10px; border-radius:999px;
      background:#fff; border:1px solid #ddd; box-shadow:0 2px 8px rgba(0,0,0,.06);
      font-size:14px;
    }
    .dot{
      width:14px; height:14px; border-radius:50%;
      box-shadow: inset 0 0 0 2px rgba(255,255,255,.6);
    }
    .dot.blue{
      background: var(--blue);
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.65) 0 2px, transparent 2px 7px);
    }
    .dot.orange{
      background: var(--orange);
      background-image:
        repeating-linear-gradient(0deg, rgba(255,255,255,.65) 0 2px, transparent 2px 7px),
        repeating-linear-gradient(90deg, rgba(255,255,255,.65) 0 2px, transparent 2px 7px);
    }

    /* Rules */
    .rules{
      margin-top:10px;
      font-size:13px; color:#444;
      background:#fff; border:1px solid #e5e5e5; border-radius:12px;
      padding:10px 12px; display:inline-block;
    }

    /* Board */
    .board-wrap{ grid-area:board; display:flex; justify-content:center; align-items:center; }
    .board{
      display:grid; grid-template-columns: repeat(3, var(--cell-size)); grid-template-rows: repeat(3, var(--cell-size));
      gap: var(--gap);
      padding: var(--gap);
      background: var(--board-bg);
      border-radius:18px;
      box-shadow:
        0 10px 30px rgba(0,0,0,.08),
        inset 0 0 0 3px #ddd;
    }
    .cell{
      position:relative;
      width:var(--cell-size); height:var(--cell-size);
      background:#fff;
      border-radius:12px;
      box-shadow: inset 0 0 0 2px #d0d0d0;
      cursor:pointer;
      transition: box-shadow .15s ease;
    }
    .cell:active{ box-shadow: inset 0 0 0 2px #bdbdbd; }

    /* Piece (top-only visual; no stack hint) */
    .piece{
      position:absolute; left:50%; top:50%; transform: translate(-50%,-50%);
      border-radius:50%;
      box-shadow: 0 6px 16px rgba(0,0,0,.18), inset 0 0 0 3px rgba(255,255,255,.65);
      transition: transform .18s ease, filter .18s ease, box-shadow .18s ease, opacity .18s ease;
      will-change: transform;
    }
    
/* 選中格子邊框（輕微加粗） */
.cell.selected {
  box-shadow: inset 0 0 0 3px #999;
}

/* 選中棋子：提亮、加粗邊框、外圈脈衝 */
.piece.selected {
  filter: brightness(1.08);
  border-width: 3px;
  transform: translate(-50%,-50%) scale(1.02);
}

/* 外圍脈衝圈（以 ::after 製作） */
.piece.selected::after {
  content: "";
  position: absolute;
  left: 50%; top: 50%; transform: translate(-50%,-50%);
  width: 105%; height: 105%;
  border-radius: 50%;
  box-shadow: 0 0 0 3px rgba(30,144,255,.35); /* 藍方光暈預設（會用 JS 覆蓋顏色） */
  animation: pulseRing 1.2s ease-in-out infinite;
  pointer-events: none;
}

/* 橙方光暈（覆蓋顏色） */
.piece.selected.orange-ring::after {
  box-shadow: 0 0 0 3px rgba(255,140,0,.35);
}

/* 選中格子邊框：稍微加粗，令位置更明顯 */
.cell.selected {
  box-shadow: inset 0 0 0 3px #888;
}

/* 選中棋子：加粗邊框 + 提亮 + 輕微放大 */
.piece.selected {
  border-width: 4px;
  filter: saturate(1.2) brightness(1.06);
  transform: translate(-50%,-50%) scale(1.05);
}

/* 強烈顏色覆蓋層（半透明） */
.piece.selected::before {
  content: "";
  position: absolute;
  left: 50%; top: 50%; transform: translate(-50%,-50%);
  width: 100%; height: 100%;
  border-radius: 50%;
  background: rgba(30,144,255, .25); /* 預設藍方覆蓋色 */
  pointer-events: none;
}

/* 橙方覆蓋層顏色 */
.piece.selected.orange-fill::before {
  background: rgba(255,140,0, .28);
}

/* 外圍光暈圈（脈衝） */
.piece.selected::after {
  content: "";
  position: absolute;
  left: 50%; top: 50%; transform: translate(-50%,-50%);
  width: 115%; height: 115%;
  border-radius: 50%;
  box-shadow: 0 0 0 4px rgba(30,144,255,.45); /* 藍方光暈 */
  animation: pulseRingStrong 1.1s ease-in-out infinite;
  pointer-events: none;
}

/* 橙方光暈 */
.piece.selected.orange-ring::after {
  box-shadow: 0 0 0 4px rgba(255,140,0,.5);
}

/* 脈衝動畫（更明顯） */
@keyframes pulseRingStrong {
  0%   { transform: translate(-50%,-50%) scale(1.00); opacity: .95; }
  50%  { transform: translate(-50%,-50%) scale(1.10); opacity: .40; }
  100% { transform: translate(-50%,-50%) scale(1.00); opacity: .95; }
}

/* 脈衝動畫 */
@keyframes pulseRing {
  0%   { transform: translate(-50%,-50%) scale(1.00); opacity: .90; }
  50%  { transform: translate(-50%,-50%) scale(1.08); opacity: .40; }
  100% { transform: translate(-50%,-50%) scale(1.00); opacity: .90; }
}

    /* sizes */
    .size-1{ width:60%; height:60%; }
    .size-2{ width:75%; height:75%; }
    .size-3{ width:90%; height:90%; }
    /* texture: 圈紋 vs 十字紋 */
    .blue-piece{
      background: var(--blue);
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.7) 0 2px, transparent 2px 8px);
      border: 2px solid #0c6fd3;
    }
    .orange-piece{
      background: var(--orange);
      background-image:
        repeating-linear-gradient(0deg, rgba(255,255,255,.7) 0 2px, transparent 2px 8px),
        repeating-linear-gradient(90deg, rgba(255,255,255,.7) 0 2px, transparent 2px 8px);
      border: 2px solid #d36a00;
    }

    /* Animations */
    @keyframes bounceIn{
      0%{ transform: translate(-50%,-50%) scale(.85); filter:brightness(1); }
      50%{ transform: translate(-50%,-50%) scale(1.06); }
      100%{ transform: translate(-50%,-50%) scale(1); }
    }
    @keyframes pressDown{
      0%{ transform: translate(-50%,-50%) scale(1); box-shadow:0 8px 20px rgba(0,0,0,.22), inset 0 0 0 3px rgba(255,255,255,.65) }
      50%{ transform: translate(-50%,-50%) scale(.96); box-shadow:0 3px 12px rgba(0,0,0,.22), inset 0 0 0 4px rgba(255,255,255,.75) }
      100%{ transform: translate(-50%,-50%) scale(1); }
    }
    @keyframes blinkRed{
      0%,100%{ color: var(--danger); opacity:1; }
      50%{ color: #ff5058; opacity:.25; }
    }
    .bounce{ animation: bounceIn .28s ease-out; }
    .press{ animation: pressDown .28s ease-out; }

    /* Trays */
    .tray{
      background:#fff; border:1px solid #e6e6e6; border-radius:14px;
      padding:12px; box-shadow: 0 6px 16px rgba(0,0,0,.06);
    }
    .tray h3{
      margin:0 0 8px; font-size:16px; display:flex; align-items:center; gap:8px;
    }
    .tray .role{ display:inline-flex; align-items:center; gap:6px; }
    .tray .role .dot{ box-shadow: inset 0 0 0 2px rgba(255,255,255,.6); }

    .tray-grid{
      display:grid; grid-template-columns: repeat(3, 1fr); gap:10px;
    }
    .tray-btn{
      display:flex; flex-direction:column; align-items:center; justify-content:center;
      gap:8px; padding:8px; cursor:pointer; border-radius:12px;
      border:1px solid #ddd; background:#fafafa; transition: all .15s ease;
      min-height:92px;
    }
    .tray-btn:hover{ background:#f5f5f5; transform: translateY(-1px); }
    .tray-btn.active{ border-color:#888; box-shadow:0 4px 12px rgba(0,0,0,.08); background:#fff; }

    .tray .mini-piece{
      border-radius:50%;
      box-shadow: 0 3px 8px rgba(0,0,0,.15), inset 0 0 0 3px rgba(255,255,255,.65);
    }
    .mini.size-1{ width:28px; height:28px; }
    .mini.size-2{ width:34px; height:34px; }
    .mini.size-3{ width:40px; height:40px; }
    .mini.blue{ background:var(--blue);
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.7) 0 2px, transparent 2px 8px);
      border:2px solid #0c6fd3;
    }
    .mini.orange{ background:var(--orange);
      background-image:
        repeating-linear-gradient(0deg, rgba(255,255,255,.7) 0 2px, transparent 2px 8px),
        repeating-linear-gradient(90deg, rgba(255,255,255,.7) 0 2px, transparent 2px 8px);
      border:2px solid #d36a00;
    }

    .count{ font-size:13px; color:#333; }
    .count.zero{ animation: blinkRed .9s infinite; font-weight:800; }

    .left{ grid-area:left; }
    .right{ grid-area:right; }

    /* Footer */
    .footer{ grid-area:footer; text-align:center; color:#666; font-size:12px; }

    /* Toast message */
    .toast{
      position:fixed; left:50%; bottom:18px; transform: translateX(-50%);
      background:#000; color:#fff; padding:10px 14px; border-radius:999px; font-size:13px;
      opacity:0; pointer-events:none; transition:opacity .18s ease, transform .18s ease;
    }
    .toast.show{ opacity:.9; transform: translateX(-50%) translateY(-4px); }
  </style>
</head>
<body>
  <div class="app">
    <!-- Header -->
    <div class="header">
      <h1 class="title">[超級過三關]</h1>
      <div class="subtitle">與朋友一起同樂 · PVP（同一裝置兩人）</div>

      <div class="controls">
        <span class="status">
          <span class="chip">
            <span>模式：</span>
            <button id="modePlace" class="btn primary">放置</button>
            <button id="modeMove" class="btn">移動</button>
          </span>
          <span class="chip">
            <span>輪到：</span>
            <span class="dot" id="turnDot"></span>
            <span id="turnText"></span>
          </span>
        </span>
        <button id="resetBtn" class="btn">重置</button>
        <button id="swapFirstBtn" class="btn">換邊先手</button>
      </div>

      <div class="rules">
        <strong>[規則重點]</strong>
        ：“放置或移動（只可移動自己最上層） • 更大的棋能覆蓋更細棋使其不能移動，除非更大的一方移開 • 勝利：只計每格最上層，先連成一線（橫/直/斜）者勝”
      </div>
    </div>

    <!-- Left Tray (Blue) -->
    <div class="left tray" id="trayBlue">
      <h3><span class="role"><span class="dot blue"></span>藍（圈紋）</span></h3>
      <div class="tray-grid">
        <div class="tray-btn" data-player="blue" data-size="3">
          <div class="mini mini-piece mini blue size-3"></div>
          <div class="count" id="count-blue-3">x 2</div>
          <div>大</div>
        </div>
        <div class="tray-btn" data-player="blue" data-size="2">
          <div class="mini mini-piece mini blue size-2"></div>
          <div class="count" id="count-blue-2">x 2</div>
          <div>中</div>
        </div>
        <div class="tray-btn" data-player="blue" data-size="1">
          <div class="mini mini-piece mini blue size-1"></div>
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

    <!-- Right Tray (Orange) -->
    <div class="right tray" id="trayOrange">
      <h3><span class="role"><span class="dot orange"></span>橙（十字紋）</span></h3>
      <div class="tray-grid">
        <div class="tray-btn" data-player="orange" data-size="3">
          <div class="mini mini-piece mini orange size-3"></div>
          <div class="count" id="count-orange-3">x 2</div>
          <div>大</div>
        </div>
        <div class="tray-btn" data-player="orange" data-size="2">
          <div class="mini mini-piece mini orange size-2"></div>
          <div class="count" id="count-orange-2">x 2</div>
          <div>中</div>
        </div>
        <div class="tray-btn" data-player="orange" data-size="1">
          <div class="mini mini-piece mini orange size-1"></div>
          <div class="count" id="count-orange-1">x 2</div>
          <div>小</div>
        </div>
      </div>
    </div>

    <!-- Footer -->
    <div class="footer">設計：更似實體棋 UI · 無提示 PVP · 動畫：彈跳/壓住 · 任何回合可放置或移動</div>

    <!-- Toast -->
    <div id="toast" class="toast" aria-live="polite"></div>
  </div>

  <script>
  (function(){
    // --- State ---
    const players = ["blue","orange"];
    const sizeNames = {1:"小",2:"中",3:"大"};
    const winLines = [
      [0,1,2],[3,4,5],[6,7,8], // rows
      [0,3,6],[1,4,7],[2,5,8], // cols
      [0,4,8],[2,4,6]          // diagonals
    ];
    let board = Array.from({length:9},()=>[]); // each cell: stack of {player,size}
    let counts = {
      blue: {1:2,2:2,3:2},
      orange: {1:2,2:2,3:2}
    };
    let current = "blue"; // default first
    let mode = "place";   // "place" or "move"
    let selectedSize = null; // for place
    let selectedFrom = null; // index for move
    let gameOver = false;

    // --- Elements ---
    const boardEl = document.getElementById("board");
    const toastEl = document.getElementById("toast");
    const modePlaceBtn = document.getElementById("modePlace");
    const modeMoveBtn  = document.getElementById("modeMove");
    const resetBtn     = document.getElementById("resetBtn");
    const swapFirstBtn = document.getElementById("swapFirstBtn");
    const turnDot      = document.getElementById("turnDot");
    const turnText     = document.getElementById("turnText");

    // Build board cells
    for(let i=0;i<9;i++){
      const c = document.createElement("div");
      c.className = "cell";
      c.dataset.index = i;
      c.addEventListener("click", ()=>onCellClick(i));
      boardEl.appendChild(c);
    }

    // Tray selection
    document.querySelectorAll(".tray-btn").forEach(btn=>{
      btn.addEventListener("click", ()=>{
        if(gameOver) return;
        const player = btn.dataset.player;
        const size   = Number(btn.dataset.size);
        // PVP 無提示：只允許當前輪到嘅一方選托盤
        if(player !== current){ showToast("未到你嗰邊喔"); return; }
        // 若庫存 0：提示並不選中
        if(counts[player][size] <= 0){
          showToast(`${playerLabel(player)} 的 ${sizeNames[size]} 已用完`);
          return;
        }
        selectedSize = size;
        selectedFrom = null;
        mode = "place";
        updateModeButtons();
        // 標示選中（非提示落點，只是托盤按鈕高亮）
        document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("active"));
        btn.classList.add("active");
      });
    });

    // Mode buttons
    modePlaceBtn.addEventListener("click", ()=>{
      if(gameOver) return;
      mode = "place";
      selectedFrom = null;
      updateModeButtons();
      showToast("模式：放置");
    });
    modeMoveBtn.addEventListener("click", ()=>{
      if(gameOver) return;
      mode = "move";
      selectedSize = null;
      clearTrayActive();
      updateModeButtons();
      showToast("模式：移動（先點選你最上層棋子）");
    });

    // Reset & Swap first
    resetBtn.addEventListener("click", ()=>resetGame(false));
    swapFirstBtn.addEventListener("click", ()=>resetGame(true));

    // --- Rendering ---

function render(){
  // Turn chip
  turnDot.className = "dot " + (current==="blue"?"blue":"orange");
  turnText.textContent = current==="blue" ? "藍" : "橙";

  // Board: render top piece only
  for(let i=0;i<9;i++){
    const cellEl = boardEl.children[i];

    // 先清除舊視覺
    const old = cellEl.querySelector(".piece");
    if(old) old.remove();
    cellEl.classList.remove("selected"); // ✅ 清除格子選中效果

    const top = topPiece(i);
    if(top){
      const p = document.createElement("div");
      p.className = `piece ${top.player==='blue'?'blue-piece':'orange-piece'} size-${top.size}`;

      // 原有動畫旗標
      if(top.justPlaced) p.classList.add("bounce");
      if(top.justPressed) p.classList.add("press");

      // ✅ 新增：如果此格是「已選中準備移動」的格子，套用選中效果
      if(selectedFrom === i && top.player === current){
        p.classList.add("selected");                 // 棋子選中樣式
        cellEl.classList.add("selected");            // 格子邊框加粗
        if(top.player === "orange") p.classList.add("orange-ring"); // 橙方光暈
      }

      cellEl.appendChild(p);

      // consume flags
      delete top.justPlaced;
      delete top.justPressed;
    }
  }

  // Trays count & blink（保持不變）
  [1,2,3].forEach(s=>{
    const cb = document.getElementById(`count-blue-${s}`);
    const co = document.getElementById(`count-orange-${s}`);
    cb.textContent = `x ${counts.blue[s]}`;
    co.textContent = `x ${counts.orange[s]}`;
    cb.classList.toggle("zero", counts.blue[s]===0);
    co.classList.toggle("zero", counts.orange[s]===0);
  });
}
``

    function updateModeButtons(){
      modePlaceBtn.classList.toggle("primary", mode==="place");
      modeMoveBtn.classList.toggle("primary", mode==="move");
    }
    function clearTrayActive(){
      document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("active"));
    }

    // --- Game actions ---

function onCellClick(index){
  if(gameOver) return;

  const tp = topPiece(index);

  // ✅ 新增：若「放置模式」但未選大小，而且你點到自己最上層棋子 → 自動進入移動模式並選中該棋
  if(mode==="place" && selectedSize===null && tp && tp.player===current){
    mode = "move";
    selectedFrom = index;
    updateModeButtons();
    showToast(`已選：第 ${index+1} 格（${sizeNames[tp.size]}）→ 再點目標格`);
    return; // 本次事件只做選中，不再往下走
  }

  if(mode==="place"){
    if(selectedSize===null){
      showToast("先喺托盤揀 大 / 中 / 小（或直接點你嘅棋子進入移動）");
      return;
    }
    if(!canPlace(current, selectedSize, index)){
      showToast("呢步唔合法（不可覆蓋同色、不可覆同大小或較大）");
      return;
    }
    placePiece(current, selectedSize, index);
    counts[current][selectedSize]--;
    // 清除托盤選中
    selectedSize = null; clearTrayActive();
    // 勝負判定
    if(checkWin(current)){
      gameOver = true;
      render();
      setTimeout(()=>alert(`勝利！${playerLabel(current)} 連成一線！`), 10);
      return;
    }
    // 換手
    switchTurn();
  }
  else if(mode==="move"){
    // Step1: 未選 from -> 先選自己最上層（保持原規則）
    if(selectedFrom===null){
      if(!tp || tp.player !== current){
        showToast("只可選你自己最上層棋子");
        return;
      }
      selectedFrom = index;
      showToast(`已選：第 ${index+1} 格（${sizeNames[tp.size]}）→ 再點目標格`);
    }else{
      // Step2: 目標
      const fromTop = topPiece(selectedFrom);
      if(!fromTop || fromTop.player !== current){
        selectedFrom = null;
        showToast("選擇失效，請重選");
        return;
      }
      if(selectedFrom===index){
        selectedFrom = null;
        showToast("已取消選擇");
        return;
      }
      if(!canMove(current, fromTop.size, selectedFrom, index)){
        showToast("移動唔合法（不可覆同色、不可覆同大小或較大）");
        return;
      }
      movePiece(current, fromTop.size, selectedFrom, index);
      selectedFrom = null;
      // 勝負判定
      if(checkWin(current)){
        gameOver = true;
        render();
        setTimeout(()=>alert(`勝利！${playerLabel(current)} 連成一線！`), 10);
        return;
      }
      // 換手
      switchTurn();
    }
  }
}

    // Place logic
    function canPlace(player,size,index){
      const stack = board[index];
      const top = stack.length ? stack[stack.length-1] : null;
      if(!top) return true; // empty
      if(top.player === player) return false; // 不可覆同色
      // 只可大吃小（嚴格大於）
      return size > top.size;
    }
    function placePiece(player,size,index){
      const stack = board[index];
      stack.push({player,size, justPlaced:true, justPressed: !!(stack.length) }); // 有覆蓋就加壓住效果
      render();
    }

    // Move logic
    function canMove(player,size,from,to){
      if(from===to) return false;
      // 只可移動自己最上層
      const fromTop = topPiece(from);
      if(!fromTop || fromTop.player!==player || fromTop.size!==size) return false;
      const toTop = topPiece(to);
      if(!toTop) return true; // 移去空格
      if(toTop.player===player) return false; // 不可覆同色
      return size > toTop.size; // 嚴格大於先可覆
    }
    function movePiece(player,size,from,to){
      const fromStack = board[from];
      const moving = fromStack.pop(); // top piece
      const toStack = board[to];
      moving.justPlaced = true; // 用同一動畫
      moving.justPressed = !!(toStack.length); // 有目標就壓住一下
      toStack.push(moving);
      render();
    }

    // Helpers
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

    function resetGame(swapFirst){
      board = Array.from({length:9},()=>[]);
      counts = { blue:{1:2,2:2,3:2}, orange:{1:2,2:2,3:2} };
      gameOver = false;
      selectedSize = null; selectedFrom = null;
      clearTrayActive();
      if(swapFirst) current = (current==="blue") ? "orange" : "blue";
      render();
      showToast(`已重置（先手：${playerLabel(current)}）`);
    }

    function playerLabel(p){ return p==="blue" ? "藍" : "橙"; }

    // Toast
    let toastTimer = null;
    function showToast(msg){
      toastEl.textContent = msg;
      toastEl.classList.add("show");
      clearTimeout(toastTimer);
      toastTimer = setTimeout(()=>toastEl.classList.remove("show"), 1200);
    }

    // Init
    render();
  })();
  </script>
</body>
</html>
