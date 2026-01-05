
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
  <title>[超級過三關] PVP / PVE</title>
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
        repeating-linear-gradient(90deg, rgba(255,255,255,.65) 0 2px, transparent 2px 8px);
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
    /* sizes */
    .size-1{ width:55%; height:55%; }
    .size-2{ width:72%; height:72%; }
    .size-3{ width:95%; height:95%; }

    .blue-piece{
      background: var(--blue);
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.70) 0 2px, transparent 2px 8px);
      border: 2px solid #0c6fd3;
    }
    .orange-piece{
      background: var(--orange);
      background-image:
        repeating-linear-gradient(0deg, rgba(255,255,255,.70) 0 2px, transparent 2px 8px),
        repeating-linear-gradient(90deg, rgba(255,255,255,.70) 0 2px, transparent 2px 8px);
      border: 2px solid #d36a00;
    }
    .blue-piece.size-1{ border-width: 2px; }
    .blue-piece.size-2{ border-width: 4px; }
    .blue-piece.size-3{ border-width: 6px; }
    .orange-piece.size-1{ border-width: 2px; }
    .orange-piece.size-2{ border-width: 4px; }
    .orange-piece.size-3{ border-width: 6px; }

    .size-badge{
      position:absolute;
      left:50%; top:50%; transform: translate(-50%,-50%);
      color:#fff; font-weight:900;
      letter-spacing:.5px;
      background: rgba(0,0,0,.35);
      border-radius:999px;
      padding: 2px 8px;
      box-shadow: 0 2px 6px rgba(0,0,0,.25);
      pointer-events:none;
      user-select:none;
    }
    .piece.size-1 .size-badge{ font-size: 12px; padding: 2px 6px; }
    .piece.size-2 .size-badge{ font-size: 14px; padding: 3px 8px; }
    .piece.size-3 .size-badge{ font-size: 16px; padding: 4px 10px; }

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

    /* 強烈選中提示 */
    .cell.selected { box-shadow: inset 0 0 0 3px #888; }
    .piece.selected {
      border-width: calc(2px + 2px);
      filter: saturate(1.2) brightness(1.06);
      transform: translate(-50%,-50%) scale(1.05);
    }
    .piece.selected::before {
      content: "";
      position: absolute;
      left: 50%; top: 50%; transform: translate(-50%,-50%);
      width: 100%; height: 100%;
      border-radius: 50%;
      background: rgba(30,144,255, .25);
      pointer-events: none;
    }
    .piece.selected.orange-fill::before { background: rgba(255,140,0, .28); }
    .piece.selected::after {
      content: "";
      position: absolute;
      left: 50%; top: 50%; transform: translate(-50%,-50%);
      width: 115%; height: 115%;
      border-radius: 50%;
      box-shadow: 0 0 0 4px rgba(30,144,255,.45);
      animation: pulseRingStrong 1.1s ease-in-out infinite;
      pointer-events: none;
    }
    .piece.selected.orange-ring::after { box-shadow: 0 0 0 4px rgba(255,140,0,.5); }
    @keyframes pulseRingStrong {
      0%   { transform: translate(-50%,-50%) scale(1.00); opacity: .95; }
      50%  { transform: translate(-50%,-50%) scale(1.10); opacity: .40; }
      100% { transform: translate(-50%,-50%) scale(1.00); opacity: .95; }
    }

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
      position: relative;
      border-radius:50%;
      box-shadow: 0 3px 8px rgba(0,0,0,.15), inset 0 0 0 3px rgba(255,255,255,.65);
    }
    .mini.size-1{ width:28px; height:28px; }
    .mini.size-2{ width:34px; height:34px; }
    .mini.size-3{ width:40px; height:40px; }
    .mini.blue{
      background:var(--blue);
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.70) 0 2px, transparent 2px 8px);
      border:2px solid #0c6fd3;
    }
    .mini.orange{
      background:var(--orange);
      background-image:
        repeating-linear-gradient(0deg, rgba(255,255,255,.70) 0 2px, transparent 2px 8px),
        repeating-linear-gradient(90deg, rgba(255,255,255,.70) 0 2px, transparent 2px 8px);
      border:2px solid #d36a00;
    }
    .mini.blue.size-1{ border-width:2px; }
    .mini.blue.size-2{ border-width:4px; }
    .mini.blue.size-3{ border-width:6px; }
    .mini.orange.size-1{ border-width:2px; }
    .mini.orange.size-2{ border-width:4px; }
    .mini.orange.size-3{ border-width:6px; }

    .mini-badge{
      position:absolute;
      left:50%; top:50%; transform: translate(-50%,-50%);
      color:#fff; font-weight:900;
      background: rgba(0,0,0,.35);
      border-radius:999px;
      padding: 1px 6px;
      font-size:12px;
      letter-spacing:.5px;
      box-shadow: 0 2px 6px rgba(0,0,0,.25);
      pointer-events:none;
      user-select:none;
    }

    .count{ font-size:13px; color:#333; }
    .count.zero{ animation: blinkRed .9s infinite; font-weight:800; }

    .left{ grid-area:left; }
    .right{ grid-area:right; }

    .footer{ grid-area:footer; text-align:center; color:#666; font-size:12px; }

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
      <div class="subtitle">與朋友一起同樂 · PVP / PVE（同一裝置）</div>

      <div class="controls">
        <span class="status">
          <span class="chip">
            <span>輪到：</span>
            <span class="dot" id="turnDot"></span>
            <span id="turnText"></span>
          </span>
        </span>

        <!-- === AI 新增：模式 / 參數控制 === -->
        <label class="chip">
          模式：
          <select id="modeSelect" style="margin-left:6px">
            <option value="pvp">玩家 vs 玩家</option>
            <option value="pve">玩家 vs 電腦</option>
          </select>
        </label>
        <label class="chip">
          AI 深度：<input id="aiDepth" type="number" min="1" max="5" value="3" style="width:60px;margin-left:6px" />
        </label>
        <label class="chip">
          預測溫度 τ：<input id="aiTau" type="number" step="0.1" min="0.1" max="5" value="1.0" style="width:70px;margin-left:6px" />
        </label>
        <label class="chip">
          學習率 η：<input id="aiLr" type="number" step="0.01" min="0.01" max="1" value="0.15" style="width:70px;margin-left:6px" />
        </label>
        <button id="resetModelBtn" class="btn">重置AI學習</button>

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
          <div class="mini mini-piece mini blue size-3">
            <span class="mini-badge">大</span>
          </div>
          <div class="count" id="count-blue-3">x 2</div>
          <div>大</div>
        </div>
        <div class="tray-btn" data-player="blue" data-size="2">
          <div class="mini mini-piece mini blue size-2">
            <span class="mini-badge">中</span>
          </div>
          <div class="count" id="count-blue-2">x 2</div>
          <div>中</div>
        </div>
        <div class="tray-btn" data-player="blue" data-size="1">
          <div class="mini mini-piece mini blue size-1">
            <span class="mini-badge">小</span>
          </div>
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
          <div class="mini mini-piece mini orange size-3">
            <span class="mini-badge">大</span>
          </div>
          <div class="count" id="count-orange-3">x 2</div>
          <div>大</div>
        </div>
        <div class="tray-btn" data-player="orange" data-size="2">
          <div class="mini mini-piece mini orange size-2">
            <span class="mini-badge">中</span>
          </div>
          <div class="count" id="count-orange-2">x 2</div>
          <div>中</div>
        </div>
        <div class="tray-btn" data-player="orange" data-size="1">
          <div class="mini mini-piece mini orange size-1">
            <span class="mini-badge">小</span>
          </div>
          <div class="count" id="count-orange-1">x 2</div>
          <div>小</div>
        </div>
      </div>
    </div>

    <!-- Footer -->
    <div class="footer">更似實體棋 · 大中小更易分辨（直徑、邊框、標籤） · PVP / PVE · 動畫：彈跳/壓住 · 同色大覆細允許 · 可隨時改為移動</div>

    <!-- Toast -->
    <div id="toast" class="toast" aria-live="polite"></div>
  </div>

  <script>
  (function(){
    // --- State ---
    const sizeNames = {1:"小",2:"中",3:"大"};
    const winLines = [
      [0,1,2],[3,4,5],[6,7,8], // rows
      [0,3,6],[1,4,7],[2,5,8], // cols
      [0,4,8],[2,4,6]          // diagonals
    ];
    let board = Array.from({length:9},()=>[]); // each cell: stack of {player,size}
    let counts = { blue:{1:2,2:2,3:2}, orange:{1:2,2:2,3:2} };
    let current = "blue";
    let selectedSize = null;   // 托盤選擇的大小（放置意圖）
    let selectedFrom = null;   // 準備移動的來源格 index
    let gameOver = false;

    // === AI 新增 ===
    let mode = "pvp";                    // 'pvp' or 'pve'
    const AI = { side: "orange" };       // 預設 AI 是橙方（可改為 "blue"）
    let aiDepth = 3;                     // 搜索深度
    let aiTau   = 1.0;                   // 預測溫度 τ
    let aiLr    = 0.15;                  // 學習率 η
    const MODEL_KEY = "super3_ai_opponent_weights_v1";
    let weights = loadWeights() || initWeights();

    // --- Elements ---
    const boardEl = document.getElementById("board");
    const toastEl = document.getElementById("toast");
    const resetBtn     = document.getElementById("resetBtn");
    const swapFirstBtn = document.getElementById("swapFirstBtn");
    const turnDot      = document.getElementById("turnDot");
    const turnText     = document.getElementById("turnText");

    // === AI 控件元素 ===
    const modeSelect     = document.getElementById("modeSelect");
    const aiDepthInput   = document.getElementById("aiDepth");
    const aiTauInput     = document.getElementById("aiTau");
    const aiLrInput      = document.getElementById("aiLr");
    const resetModelBtn  = document.getElementById("resetModelBtn");

    // Build board cells
    for(let i=0;i<9;i++){
      const c = document.createElement("div");
      c.className = "cell";
      c.dataset.index = i;
      c.addEventListener("click", ()=>onCellClick(i));
      boardEl.appendChild(c);
    }

    // Tray selection（支援改變主意：再次點同一按鈕可取消；也可點棋盤移動）
    document.querySelectorAll(".tray-btn").forEach(btn=>{
      btn.addEventListener("click", ()=>{
        if(gameOver) return;
        const player = btn.dataset.player;
        const size   = Number(btn.dataset.size);
        const isActive = btn.classList.contains("active");

        if(player !== current){ showToast("未到你嗰邊喔"); return; }
        if(counts[player][size] <= 0){
          showToast(`${playerLabel(player)} 的 ${sizeNames[size]} 已用完`);
          return;
        }

        // PvE: 禁止 AI 回合的人為操作
        if(mode === "pve" && current === AI.side){
          showToast("AI 正在思考中…");
          return;
        }

        // 再次點同一個已選按鈕 → 取消出棋
        if(isActive && selectedSize === size){
          selectedSize = null;
          btn.classList.remove("active");
          showToast("已取消出棋，可改為移動（點棋盤內自己棋子）");
          return;
        }

        // 揀新大小（進入放置意圖）
        selectedSize = size;
        // 改為出棋時，清除任何移動選擇
        if(selectedFrom !== null){ selectedFrom = null; render(); }
        // 托盤按鈕高亮
        document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("active"));
        btn.classList.add("active");
        showToast(`已選擇：${playerLabel(player)} 的${sizeNames[size]}（點棋盤落子；如要改為移動，可重按托盤取消）`);
      });
    });

    // Reset & Swap first
    resetBtn.addEventListener("click", ()=>resetGame(false));
    swapFirstBtn.addEventListener("click", ()=>resetGame(true));

    // === AI 控制事件 ===
    modeSelect.addEventListener("change", ()=>{
      mode = modeSelect.value;
      render();
      if(mode === "pve" && current === AI.side){
        // 若換成 PvE 且正好輪到 AI → 立即思考
        setTimeout(aiPlay, 150);
      }
    });
    aiDepthInput.addEventListener("change", ()=>{ aiDepth = clamp(parseInt(aiDepthInput.value||"3",10),1,5); aiDepthInput.value = aiDepth; });
    aiTauInput.addEventListener("change", ()=>{ aiTau = clamp(parseFloat(aiTauInput.value||"1.0"),0.1,5); aiTauInput.value = aiTau.toFixed(2); });
    aiLrInput.addEventListener("change", ()=>{ aiLr = clamp(parseFloat(aiLrInput.value||"0.15"),0.01,1); aiLrInput.value = aiLr.toFixed(2); });
    resetModelBtn.addEventListener("click", ()=>{ resetWeights(); alert("AI 對手模型已重置"); });

    // --- Rendering ---
    function ensureSelectionValid(){
      if(selectedFrom !== null){
        const t = topPiece(selectedFrom);
        if(!t || t.player !== current){
          selectedFrom = null;
        }
      }
    }

    function render(){
      ensureSelectionValid();

      // Turn chip
      turnDot.className = "dot " + (current==="blue"?"blue":"orange");
      const aiFlag = (mode==="pve" && current===AI.side) ? "（AI）" : "";
      turnText.textContent = (current==="blue" ? "藍" : "橙") + aiFlag;

      // Board: render top piece only（無堆疊視覺）
      for(let i=0;i<9;i++){
        const cellEl = boardEl.children[i];
        const old = cellEl.querySelector(".piece");
        if(old) old.remove();
        cellEl.classList.remove("selected");

        const top = topPiece(i);
        if(top){
          const p = document.createElement("div");
          p.className = `piece ${top.player==='blue'?'blue-piece':'orange-piece'} size-${top.size}`;

          // 中央尺寸標籤
          const badge = document.createElement("span");
          badge.className = "size-badge";
          badge.textContent = sizeNames[top.size];
          p.appendChild(badge);

          // 動畫旗標
          if(top.justPlaced) p.classList.add("bounce");
          if(top.justPressed) p.classList.add("press");

          // 明顯顏色提示：被選中準備移動的棋子
          if(selectedFrom === i && top.player === current){
            p.classList.add("selected");
            cellEl.classList.add("selected");
            if(top.player === "orange"){
              p.classList.add("orange-fill");
              p.classList.add("orange-ring");
            }
          }

          cellEl.appendChild(p);
          delete top.justPlaced;
          delete top.justPressed;
        }
      }

      // Trays count & blink
      [1,2,3].forEach(s=>{
        const cb = document.getElementById(`count-blue-${s}`);
        const co = document.getElementById(`count-orange-${s}`);
        cb.textContent = `x ${counts.blue[s]}`;
        co.textContent = `x ${counts.orange[s]}`;
        cb.classList.toggle("zero", counts.blue[s]===0);
        co.classList.toggle("zero", counts.orange[s]===0);
      });
    }

    function clearTrayActive(){
      document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("active"));
    }

    // --- Game actions ---
    function onCellClick(index){
      if(gameOver) return;

      // PvE：AI 的回合禁止玩家操作
      if(mode === "pve" && current === AI.side){
        showToast("AI 正在思考中…");
        return;
      }

      const tp = topPiece(index);

      // 1) 已在托盤選了大小 → 嘗試放置（允許同色大覆細）
      if(selectedSize !== null){
        if(!canPlace(current, selectedSize, index)){
          showToast("呢步唔合法（不可覆同大小或較大）");
          return;
        }
        const prevState = cloneState(); // *** 給 AI 學習用
        const actionObj = {type:"place", player:current, size:selectedSize, to:index};
        placePiece(current, selectedSize, index);
        counts[current][selectedSize]--;
        selectedSize = null; clearTrayActive();

        // PvE：人類（藍方）動作 → AI 線上學習
        if(mode==="pve" && current!==AI.side){
          learnFromActual(prevState, actionObj);
        }

        if(checkWin(current)){
          gameOver = true;
          render();
          setTimeout(()=>alert(`勝利！${playerLabel(current)} 連成一線！`), 10);
          return;
        }
        switchTurn();
        return;
      }

      // 2) 未選大小 → 移動流程
      // Step1: 未選來源格 → 點自己最上層棋子以選中
      if(selectedFrom === null){
        if(!tp || tp.player !== current){
          showToast("只可選你自己最上層棋子（或去托盤揀大小落子）");
          return;
        }
        selectedFrom = index;
        render(); // 顯示選中高亮
        showToast(`已選：第 ${index+1} 格（${sizeNames[tp.size]}）→ 再點目標格`);
        return;
      }

      // Step2: 已選來源格 → 嘗試移動（包括同色大覆細）
      const fromTop = topPiece(selectedFrom);

      // 選擇失效（例如被覆蓋）→ 清除選中
      if(!fromTop || fromTop.player !== current){
        selectedFrom = null;
        render();
        showToast("選擇失效，請重選");
        return;
      }

      // 點同一格 → 取消選擇
      if(selectedFrom === index){
        selectedFrom = null;
        render();
        showToast("已取消選擇");
        return;
      }

      // 嘗試移動到目標格（允許覆蓋同色較小）
      if(!canMove(current, fromTop.size, selectedFrom, index)){
        // 不合法移動：保留原選中，不切換、不取消
        showToast("移動唔合法（不可覆同大小或較大）");
        return;
      }

      // 合法移動
      const prevState = cloneState(); // *** 給 AI 學習用
      const actionObj = {type:"move", player:current, size:fromTop.size, from:selectedFrom, to:index};
      movePiece(current, fromTop.size, selectedFrom, index);
      selectedFrom = null;

      // PvE：人類（藍方）動作 → AI 線上學習
      if(mode==="pve" && current!==AI.side){
        learnFromActual(prevState, actionObj);
      }

      if(checkWin(current)){
        gameOver = true;
        render();
        setTimeout(()=>alert(`勝利！${playerLabel(current)} 連成一線！`), 10);
        return;
      }
      switchTurn();
    }

    // Place logic（允許同色大覆細）
    function canPlace(player,size,index){
      const stack = board[index];
      const top = stack.length ? stack[stack.length-1] : null;
      if(!top) return true;            // 空格可放
      return size > top.size;          // 無論同色或異色，只能大吃小
    }
    function placePiece(player,size,index){
      const stack = board[index];
      stack.push({player,size, justPlaced:true, justPressed: !!(stack.length) }); // 覆蓋→壓住效果
      render();
      // PvE：AI自動下棋（若輪到AI）
      if(mode==="pve" && current===AI.side && !gameOver) setTimeout(aiPlay, 220);
    }

    // Move logic（允許同色大覆細）
    function canMove(player,size,from,to){
      if(from===to) return false;
      const fromTop = topPiece(from);
      if(!fromTop || fromTop.player!==player || fromTop.size!==size) return false; // 只可移動自己最上層
      const toTop = topPiece(to);
      if(!toTop) return true;          // 移去空格
      return size > toTop.size;        // 嚴格大於先可覆（同色或異色皆可）
    }
    function movePiece(player,size,from,to){
      const fromStack = board[from];
      const moving = fromStack.pop(); // top piece
      const toStack = board[to];
      moving.justPlaced = true;       // 用同一彈跳動畫
      moving.justPressed = !!(toStack.length); // 有目標堆疊→壓住一下
      toStack.push(moving);
      render();
      // PvE：AI自動下棋（若輪到AI）
      if(mode==="pve" && current===AI.side && !gameOver) setTimeout(aiPlay, 220);
    }

    // Helpers
    function topPiece(index){ const s = board[index]; return s.length ? s[s.length-1] : null; }
    function playerLabel(p){ return p==="blue" ? "藍" : "橙"; }

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
      // PvE：若重置後是 AI 先手 → 立即思考
      if(mode==="pve" && current===AI.side) setTimeout(aiPlay, 200);
    }

    function clearTrayActive(){
      document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("active"));
    }

    // Toast
    let toastTimer = null;
    function showToast(msg){
      toastEl.textContent = msg;
      toastEl.classList.add("show");
      clearTimeout(toastTimer);
      toastTimer = setTimeout(()=>toastEl.classList.remove("show"), 1200);
    }

    // === AI：對手模型 + 搜索整合 ===

    function initWeights(){
      // 5 個特徵權重： [立即勝, 阻擋對手立即勝, 中心, 兩連威脅, 覆蓋對手]
      return [0.9, 0.7, 0.25, 0.55, 0.45];
    }
    function saveWeights(w){ localStorage.setItem(MODEL_KEY, JSON.stringify(w)); }
    function loadWeights(){
      const raw = localStorage.getItem(MODEL_KEY);
      return raw ? JSON.parse(raw) : null;
    }
    function resetWeights(){
      weights = initWeights();
      saveWeights(weights);
    }

    // --- 純狀態工具（供搜索與學習用，不碰 DOM） ---
    function cloneState(){
      const b = board.map(stack => stack.map(p => ({player:p.player, size:p.size})));
      const c = { blue:{1:counts.blue[1],2:counts.blue[2],3:counts.blue[3]},
                  orange:{1:counts.orange[1],2:counts.orange[2],3:counts.orange[3]} };
      return { board:b, counts:c, current };
    }
    function topAt(state, idx){
      const s = state.board[idx];
      return s.length ? s[s.length-1] : null;
    }
    function isWinState(state, player){
      return winLines.some(line => line.every(i => {
        const t = topAt(state, i);
        return t && t.player === player;
      }));
    }

    function canPlaceState(state, player, size, idx){
      const top = topAt(state, idx);
      if(!top) return true;
      return size > top.size;
    }
    function canMoveState(state, player, size, from, to){
      if(from===to) return false;
      const fromTop = topAt(state, from);
      if(!fromTop || fromTop.player!==player || fromTop.size!==size) return false;
      const toTop = topAt(state, to);
      if(!toTop) return true;
      return size > toTop.size;
    }
    function applyActionState(state, action){
      const ns = {
        board: state.board.map(stack => stack.map(p => ({player:p.player, size:p.size}))),
        counts: { blue:{...state.counts.blue}, orange:{...state.counts.orange} },
        current: state.current
      };
      if(action.type === "place"){
        ns.board[action.to].push({player:action.player, size:action.size});
        ns.counts[action.player][action.size]--;
      }else{
        const moving = ns.board[action.from].pop();
        ns.board[action.to].push(moving);
      }
      ns.current = (ns.current==="blue") ? "orange" : "blue";
      return ns;
    }

    function legalActions(state, player){
      const acts = [];

      // 放置
      [1,2,3].forEach(s=>{
        if(state.counts[player][s] > 0){
          for(let idx=0; idx<9; idx++){
            if(canPlaceState(state, player, s, idx)){
              acts.push({type:"place", player, size:s, to:idx});
            }
          }
        }
      });

      // 移動（只可移動自己最上層）
      for(let from=0; from<9; from++){
        const t = topAt(state, from);
        if(!t || t.player!==player) continue;
        const sz = t.size;
        for(let to=0; to<9; to++){
          if(from===to) continue;
          if(canMoveState(state, player, sz, from, to)){
            acts.push({type:"move", player, size:sz, from, to});
          }
        }
      }
      return acts;
    }

    function immediateWinCount(state, player){
      const acts = legalActions(state, player);
      let cnt = 0;
      for(const a of acts){
        const ns = applyActionState(state, a);
        if(isWinState(ns, player)) cnt++;
      }
      return cnt;
    }

    // 特徵：φ(state, action, player, opp)
    function featuresOnAction(state, action, player){
      const opp = (player==="blue") ? "orange" : "blue";
      const next = applyActionState(state, action);

      // 1) 立即勝
      const f_winNow = isWinState(next, player) ? 1 : 0;

      // 2) 阻擋對手立即勝（差分）
      const oppWinsBefore = immediateWinCount(state, opp);
      const oppWinsAfter  = immediateWinCount(next,  opp);
      const f_block = Math.max(0, oppWinsBefore - oppWinsAfter);

      // 3) 中心控制（落子/落到中心）
      const center = 4;
      const toIdx = (action.type==="place") ? action.to : action.to;
      const f_center = (toIdx === center) ? 1 : 0;

      // 4) 兩連威脅（只看頂層）
      let f_two = 0;
      for(const line of winLines){
        const cs = line.map(i=>topAt(next,i));
        const pc = cs.filter(t=>t && t.player===player).length;
        const oc = cs.filter(t=>t && t.player===opp).length;
        if(oc===0 && pc===2) f_two += 1;
      }

      // 5) 覆蓋對手（若目標頂層存在且是對手）
      let f_coverOpp = 0;
      if(action.type==="place"){
        const topBefore = topAt(state, action.to);
        if(topBefore && topBefore.player===opp && action.size>topBefore.size) f_coverOpp = 1;
      }else{
        const topBefore = topAt(state, action.to);
        if(topBefore && topBefore.player===opp && action.size>topBefore.size) f_coverOpp = 1;
      }

      return [f_winNow, f_block, f_center, f_two, f_coverOpp];
    }

    function softmax(xs, t=1.0){
      const m = Math.max(...xs);
      const exps = xs.map(v => Math.exp((v - m) / t));
      const s = exps.reduce((a,b)=>a+b, 0);
      return exps.map(v => v/s);
    }
    function dot(a,b){ return a.reduce((s, v, i) => s + v*b[i], 0); }

    // 對手預測（預測「藍方」在此局面下最可能的行動）
    function predictOpponent(state){
      const player = (AI.side==="orange") ? "blue" : "orange"; // 模型鎖定對手（人類）
      const acts = legalActions(state, player);
      if(acts.length === 0) return {moves:[], probs:[]};
      const scores = acts.map(a => dot(weights, featuresOnAction(state, a, player)));
      const probs  = softmax(scores, aiTau);
      return { moves: acts, probs };
    }

    // 線上學習（負對數似然的近似梯度）
    function learnFromActual(prevState, actualAction){
      // 只在 PvE，且人類（對手 AI 的一方）行動時學習
      const opponent = (AI.side==="orange") ? "blue" : "orange";
      if(actualAction.player !== opponent) return;

      const pred = predictOpponent(prevState);
      const {moves, probs} = pred;
      if(moves.length===0) return;

      // Δw = η * (φ(actual) − Σ p(m) φ(m))
      const phiActual = featuresOnAction(prevState, actualAction, opponent);
      const expectedPhi = new Array(weights.length).fill(0);
      for(let i=0;i<moves.length;i++){
        const phi = featuresOnAction(prevState, moves[i], opponent);
        for(let k=0;k<weights.length;k++){
          expectedPhi[k] += probs[i] * phi[k];
        }
      }
      for(let k=0;k<weights.length;k++){
        weights[k] += aiLr * (phiActual[k] - expectedPhi[k]);
      }
      saveWeights(weights);
    }

    // 評估函數（非終局時）
    function evaluateState(state, aiPlayer){
      const opp = (aiPlayer==="blue") ? "orange" : "blue";
      if(isWinState(state, aiPlayer)) return 10000;
      if(isWinState(state, opp)) return -10000;

      let score = 0;
      // 中心佔領
      const tCenter = topAt(state,4);
      if(tCenter){
        if(tCenter.player===aiPlayer) score += 12;
        else score -= 12;
      }
      // 兩連威脅
      for(const line of winLines){
        const cs = line.map(i=>topAt(state,i));
        const pc = cs.filter(t=>t && t.player===aiPlayer).length;
        const oc = cs.filter(t=>t && t.player===opp).length;
        if(oc===0) score += pc*pc*5;  // 我方連線加分（平方）
        if(pc===0) score -= oc*oc*5;  // 對方連線扣分
      }
      // 可覆蓋機會（潛在吃子）
      let coverChances = 0;
      const acts = legalActions(state, aiPlayer);
      for(const a of acts){
        const toIdx = (a.type==="place") ? a.to : a.to;
        const topBefore = topAt(state, toIdx);
        if(topBefore && topBefore.player===opp && a.size>topBefore.size) coverChances++;
      }
      score += coverChances * 3;

      return score;
    }

    // Minimax + α-β；對手節點用「預測排序」
    function minimax(state, depth, isMax, aiPlayer, alpha, beta){
      const opp = (aiPlayer==="blue") ? "orange" : "blue";

      if(isWinState(state, aiPlayer)) return 10000 + depth;
      if(isWinState(state, opp))      return -10000 - depth;
      if(depth===0) return evaluateState(state, aiPlayer);

      if(isMax){
        let v = -Infinity;
        const acts = legalActions(state, aiPlayer);
        for(const a of acts){
          const ns = applyActionState(state, a);
          v = Math.max(v, minimax(ns, depth-1, false, aiPlayer, alpha, beta));
          alpha = Math.max(alpha, v);
          if(beta <= alpha) break;
        }
        return v;
      }else{
        // 對手節點：按 OpponentModel 機率排序，僅擴展前 K（降分支）
        const pred = predictOpponent(state);
        const order = pred.moves.map((m,i)=>({m, p:pred.probs[i]}))
                                .sort((a,b)=> b.p - a.p);
        const K = Math.min(6, order.length);
        let v = Infinity;
        for(let i=0;i<K;i++){
          const a = order[i].m;
          const ns = applyActionState(state, a);
          v = Math.min(v, minimax(ns, depth-1, true, aiPlayer, alpha, beta));
          beta = Math.min(beta, v);
          if(beta <= alpha) break;
        }
        return v;
      }
    }

    function aiChooseAction(state, aiPlayer, maxDepth){
      let best = null, bestVal = -Infinity;
      const acts = legalActions(state, aiPlayer);
      if(acts.length===0) return null;
      for(const a of acts){
        const ns = applyActionState(state, a);
        const val = minimax(ns, maxDepth-1, false, aiPlayer, -Infinity, Infinity);
        if(val > bestVal){ bestVal = val; best = a; }
      }
      return best;
    }

    function applyActionToGlobals(action){
      if(action.type==="place"){
        placePiece(action.player, action.size, action.to);
        counts[action.player][action.size]--;
      }else{
        movePiece(action.player, action.size, action.from, action.to);
      }
    }

    function aiPlay(){
      if(gameOver || mode!=="pve") return;
      if(current !== AI.side) return;
      // 思考
      const state = cloneState();
      const action = aiChooseAction(state, AI.side, aiDepth);
      if(!action){
        // 沒合法步（理論上極少見）→ 切回人類
        current = (current==="blue") ? "orange" : "blue";
        render();
        return;
      }
      applyActionToGlobals(action);

      // 勝負判定
      const aiWon = checkWin(AI.side);
      if(aiWon){
        gameOver = true;
        render();
        setTimeout(()=>alert(`勝利！${playerLabel(AI.side)}（AI）連成一線！`), 10);
        return;
      }
      // 切回人類
      switchTurn();
    }

    function clamp(v,min,max){ return Math.max(min, Math.min(max, v)); }

    // Init
    render();
  })();
  </script>
</body>
</html>
