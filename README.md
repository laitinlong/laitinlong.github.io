
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
  <title>[超級過三關] PVP · 豪華暗色主題</title>
  <style>
    :root{
      /* 主題色 */
      --bg-deep-1:#0b0f19;
      --bg-deep-2:#121826;
      --panel:#0f1422;
      --panel-2:#131a2b;
      --glass:#0f172a80; /* 半透明玻璃 */
      --gold:#d4af37;
      --gold-soft:#e6c55a;
      --text:#e8edf5;
      --muted:#9aa5b1;
      --danger:#ff5468;
      --blue:#3aa0ff;   /* 亮藍 */
      --orange:#ff9a2f; /* 亮橙 */

      /* 尺寸 */
      --cell-size: min(22vmin, 130px);
      --gap: 12px;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0;
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans TC", "Microsoft JhengHei", Arial, sans-serif;
      color:var(--text);
      /* 奢華深色漸層背景 + 星點 */
      background:
        radial-gradient(1200px 700px at 10% 0%, #162039 0%, transparent 65%),
        radial-gradient(900px 600px at 80% 100%, #1b2544 0%, transparent 70%),
        linear-gradient(180deg, var(--bg-deep-1), var(--bg-deep-2));
      display:flex; min-height:100vh; align-items:center; justify-content:center;
      padding:18px;
      position:relative;
      overflow:hidden;
    }
    /* 星點輕微動態 */
    body::before{
      content:"";
      position:fixed; inset:0;
      background-image:
        radial-gradient(1px 1px at 20% 30%, rgba(255,255,255,.10), transparent 60%),
        radial-gradient(1px 1px at 65% 70%, rgba(255,255,255,.08), transparent 60%),
        radial-gradient(1px 1px at 35% 80%, rgba(255,255,255,.06), transparent 60%),
        radial-gradient(1px 1px at 85% 20%, rgba(255,255,255,.08), transparent 60%);
      opacity:.35; pointer-events:none;
      animation: twinkle 6s ease-in-out infinite alternate;
    }
    @keyframes twinkle{
      0%{opacity:.25; transform:scale(1)}
      100%{opacity:.45; transform:scale(1.02)}
    }

    .app{
      width:100%;
      max-width:1200px;
      display:grid;
      grid-template-columns: 1fr minmax(300px, 520px) 1fr;
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
      font-weight:800; letter-spacing:.8px;
      font-size: clamp(22px, 5vw, 40px);
      margin:0;
      text-shadow: 0 8px 30px rgba(212,175,55,.15), 0 2px 8px rgba(0,0,0,.45);
    }
    .subtitle{
      font-size: clamp(13px, 2.5vw, 16px);
      color:var(--muted);
      margin-top:6px;
    }

    .controls{
      margin-top:12px;
      display:flex; flex-wrap:wrap; gap:10px; justify-content:center; align-items:center;
    }
    .btn{
      border:1px solid rgba(212,175,55,.35);
      background: linear-gradient(180deg, #151a2a, #0f1422);
      color:var(--text); padding:9px 14px; border-radius:12px;
      cursor:pointer; transition: all .15s ease;
      box-shadow: 0 4px 14px rgba(212,175,55,.08), inset 0 0 0 1px rgba(255,255,255,.06);
    }
    .btn:hover{ transform: translateY(-1px); box-shadow:0 10px 24px rgba(212,175,55,.15); }
    .btn:active{ transform: translateY(1px); }

    .status{
      font-weight:700;
      display:inline-flex; align-items:center; gap:8px;
      padding:8px 10px; border-radius:14px;
      background: linear-gradient(180deg, #131a2b, #0f1422);
      border:1px solid rgba(212,175,55,.35);
      box-shadow: 0 4px 16px rgba(212,175,55,.12), inset 0 0 0 1px rgba(255,255,255,.05);
    }
    .chip{
      display:inline-flex; align-items:center; gap:8px; padding:6px 10px; border-radius:999px;
      background: rgba(15,20,34,.65); border:1px solid rgba(212,175,55,.35);
      box-shadow: 0 2px 12px rgba(0,0,0,.35), inset 0 0 0 1px rgba(255,255,255,.05);
      font-size:14px;
    }
    .dot{
      width:16px; height:16px; border-radius:50%;
      box-shadow: 0 0 12px rgba(212,175,55,.22), inset 0 0 0 2px rgba(255,255,255,.25);
    }
    .dot.blue{
      background: var(--blue);
      background-image: radial-gradient(circle at 50% 50%, rgba(255,255,255,.65) 0 2px, transparent 2px 7px);
      box-shadow: 0 0 12px rgba(58,160,255,.65), inset 0 0 0 2px rgba(255,255,255,.25);
    }
    .dot.orange{
      background: var(--orange);
      background-image:
        repeating-linear-gradient(0deg, rgba(255,255,255,.65) 0 2px, transparent 2px 7px),
        repeating-linear-gradient(90deg, rgba(255,255,255,.65) 0 2px, transparent 2px 7px);
      box-shadow: 0 0 12px rgba(255,154,47,.65), inset 0 0 0 2px rgba(255,255,255,.25);
    }

    /* Rules */
    .rules{
      margin-top:12px;
      font-size:13px; color:#cbd5e1;
      background: linear-gradient(180deg, #121826, #0f1422);
      border:1px solid rgba(212,175,55,.35); border-radius:14px;
      padding:12px 14px; display:inline-block;
      box-shadow: 0 6px 20px rgba(212,175,55,.12), inset 0 0 0 1px rgba(255,255,255,.06);
    }

    /* Board */
    .board-wrap{ grid-area:board; display:flex; justify-content:center; align-items:center; }
    .board{
      display:grid; grid-template-columns: repeat(3, var(--cell-size)); grid-template-rows: repeat(3, var(--cell-size));
      gap: var(--gap);
      padding: var(--gap);
      background: linear-gradient(180deg, rgba(19,26,43,.85), rgba(15,20,34,.85));
      backdrop-filter: blur(8px);
      border-radius:20px;
      border:1px solid rgba(212,175,55,.45);
      box-shadow:
        0 18px 40px rgba(0,0,0,.45),
        inset 0 0 0 2px rgba(255,255,255,.05),
        0 0 35px rgba(212,175,55,.18);
    }
    .cell{
      position:relative;
      width:var(--cell-size); height:var(--cell-size);
      background: radial-gradient(120% 120% at 30% 20%, #192033 0%, #0f1422 65%);
      border-radius:14px;
      border:1px solid rgba(212,175,55,.35);
      box-shadow:
        inset 0 0 0 1px rgba(255,255,255,.05),
        0 6px 18px rgba(0,0,0,.45);
      cursor:pointer;
      transition: box-shadow .15s ease, transform .15s ease;
    }
    .cell:hover{ box-shadow: 0 12px 24px rgba(212,175,55,.15), inset 0 0 0 1px rgba(255,255,255,.08); transform: translateY(-1px); }
    .cell:active{ transform: translateY(1px); }

    /* Piece (top-only visual) */
    .piece{
      position:absolute; left:50%; top:50%; transform: translate(-50%,-50%);
      border-radius:50%;
      box-shadow: 0 12px 28px rgba(0,0,0,.55), inset 0 0 0 3px rgba(255,255,255,.45);
      transition: transform .18s ease, filter .18s ease, box-shadow .18s ease, opacity .18s ease;
      will-change: transform;
    }
    /* sizes: 更大差距的直徑 */
    .size-1{ width:55%; height:55%; }
    .size-2{ width:72%; height:72%; }
    .size-3{ width:95%; height:95%; }

    /* textures: 藍圈 vs 橙十字 */
    .blue-piece{
      background:
        radial-gradient(110% 110% at 30% 25%, #5bb6ff 0%, #2585e6 40%, #1e6cc3 72%, #15539a 100%);
      /* 圈紋 */
      background-blend-mode: overlay;
      border: 2px solid #87c8ff;
      box-shadow: 0 0 22px rgba(58,160,255,.55), inset 0 0 0 2px rgba(255,255,255,.45);
    }
    .blue-piece::after{
      /* 圈紋紋理 */
      content:"";
      position:absolute; inset:0;
      border-radius:50%;
      background-image: repeating-radial-gradient(circle at 50% 50%, rgba(255,255,255,.55) 0 2px, transparent 2px 8px);
      opacity:.35; pointer-events:none;
    }
    .orange-piece{
      background:
        radial-gradient(110% 110% at 30% 25%, #ffc078 0%, #ff9f3f 40%, #f27f1a 70%, #c86506 100%);
      /* 十字紋 */
      background-blend-mode: overlay;
      border: 2px solid #ffd79f;
      box-shadow: 0 0 22px rgba(255,154,47,.55), inset 0 0 0 2px rgba(255,255,255,.45);
    }
    .orange-piece::after{
      content:"";
      position:absolute; inset:0;
      border-radius:50%;
      background-image:
        repeating-linear-gradient(0deg, rgba(255,255,255,.6) 0 2px, transparent 2px 8px),
        repeating-linear-gradient(90deg, rgba(255,255,255,.6) 0 2px, transparent 2px 8px);
      opacity:.25; pointer-events:none;
    }
    /* 邊框粗度隨大小改變（更易分辨） */
    .blue-piece.size-1{ border-width: 2px; }
    .blue-piece.size-2{ border-width: 4px; }
    .blue-piece.size-3{ border-width: 6px; }
    .orange-piece.size-1{ border-width: 2px; }
    .orange-piece.size-2{ border-width: 4px; }
    .orange-piece.size-3{ border-width: 6px; }

    /* 中央尺寸標籤：小／中／大 */
    .size-badge{
      position:absolute;
      left:50%; top:50%; transform: translate(-50%,-50%);
      color:#fff; font-weight:900;
      letter-spacing:.5px;
      background: linear-gradient(180deg, rgba(0,0,0,.55), rgba(20,24,34,.65));
      border:1px solid rgba(212,175,55,.55);
      border-radius:999px;
      padding: 2px 8px;
      box-shadow: 0 4px 10px rgba(0,0,0,.45), 0 0 12px rgba(212,175,55,.25);
      pointer-events:none;
      user-select:none;
      text-shadow: 0 1px 2px rgba(0,0,0,.6);
    }
    .piece.size-1 .size-badge{ font-size: 12px; padding: 2px 6px; }
    .piece.size-2 .size-badge{ font-size: 14px; padding: 3px 8px; }
    .piece.size-3 .size-badge{ font-size: 16px; padding: 4px 10px; }

    /* Animations */
    @keyframes bounceIn{
      0%{ transform: translate(-50%,-50%) scale(.85); filter:brightness(1); }
      50%{ transform: translate(-50%,-50%) scale(1.08); }
      100%{ transform: translate(-50%,-50%) scale(1); }
    }
    @keyframes pressDown{
      0%{ transform: translate(-50%,-50%) scale(1); box-shadow:0 20px 40px rgba(0,0,0,.55), inset 0 0 0 3px rgba(255,255,255,.65) }
      50%{ transform: translate(-50%,-50%) scale(.96); box-shadow:0 10px 24px rgba(0,0,0,.55), inset 0 0 0 4px rgba(255,255,255,.75) }
      100%{ transform: translate(-50%,-50%) scale(1); }
    }
    @keyframes blinkRed{
      0%,100%{ color: var(--danger); opacity:1; }
      50%{ color: #ff7b8d; opacity:.28; }
    }
    .bounce{ animation: bounceIn .30s ease-out; }
    .press{ animation: pressDown .28s ease-out; }

    /* 選中顏色提示（豪華霓虹 + 金色格框） */
    .cell.selected {
      box-shadow:
        0 0 0 2px rgba(212,175,55,.75),
        0 12px 24px rgba(212,175,55,.20),
        inset 0 0 0 2px rgba(255,255,255,.08);
      border-color: rgba(212,175,55,.85);
    }
    .piece.selected{
      filter: saturate(1.25) brightness(1.06);
      transform: translate(-50%,-50%) scale(1.06);
      box-shadow:
        0 0 18px rgba(255,255,255,.25),
        0 0 36px rgba(212,175,55,.35),
        inset 0 0 0 3px rgba(255,255,255,.75);
    }
    /* 顏色覆蓋層（半透明） */
    .piece.selected::before{
      content:"";
      position:absolute; left:50%; top:50%; transform: translate(-50%,-50%);
      width:100%; height:100%;
      border-radius:50%;
      background: rgba(58,160,255,.25); /* default 藍 */
      pointer-events:none;
    }
    .piece.selected.orange-fill::before{ background: rgba(255,154,47,.28); }
    /* 霓虹外圈 */
    .piece.selected::after{
      content:"";
      position:absolute; left:50%; top:50%; transform: translate(-50%,-50%);
      width:118%; height:118%;
      border-radius:50%;
      box-shadow: 0 0 0 5px rgba(58,160,255,.55), 0 0 40px rgba(58,160,255,.55);
      animation: pulseRingStrong 1.05s ease-in-out infinite;
      pointer-events:none;
    }
    .piece.selected.orange-ring::after{
      box-shadow: 0 0 0 5px rgba(255,154,47,.6), 0 0 40px rgba(255,154,47,.55);
    }
    @keyframes pulseRingStrong{
      0%{ transform: translate(-50%,-50%) scale(1.00); opacity:.95; }
      50%{ transform: translate(-50%,-50%) scale(1.12); opacity:.40; }
      100%{ transform: translate(-50%,-50%) scale(1.00); opacity:.95; }
    }

    /* Tray */
    .tray{
      background: linear-gradient(180deg, rgba(19,26,43,.85), rgba(15,20,34,.85));
      backdrop-filter: blur(8px);
      border:1px solid rgba(212,175,55,.45); border-radius:16px;
      padding:14px; box-shadow: 0 18px 40px rgba(0,0,0,.45), 0 0 35px rgba(212,175,55,.18);
    }
    .tray h3{
      margin:0 0 10px; font-size:16px; display:flex; align-items:center; gap:8px;
      color: var(--gold-soft);
      text-shadow: 0 2px 10px rgba(212,175,55,.25);
    }
    .tray .role{ display:inline-flex; align-items:center; gap:6px; }

    .tray-grid{
      display:grid; grid-template-columns: repeat(3, 1fr); gap:12px;
    }
    .tray-btn{
      display:flex; flex-direction:column; align-items:center; justify-content:center;
      gap:8px; padding:10px; cursor:pointer; border-radius:14px;
      border:1px solid rgba(212,175,55,.35);
      background: radial-gradient(120% 120% at 30% 20%, #192033 0%, #0f1422 65%);
      transition: all .15s ease;
      min-height:100px;
      box-shadow: inset 0 0 0 1px rgba(255,255,255,.06), 0 6px 18px rgba(0,0,0,.35);
    }
    .tray-btn:hover{ transform: translateY(-1px); box-shadow:0 12px 24px rgba(212,175,55,.15); }
    .tray-btn.active{
      border-color: rgba(212,175,55,.85);
      box-shadow: 0 10px 28px rgba(212,175,55,.22), inset 0 0 0 1px rgba(255,255,255,.08);
      background: radial-gradient(120% 120% at 30% 20%, #223052 0%, #131a2b 65%);
    }

    .tray .mini-piece{
      position: relative;
      border-radius:50%;
      box-shadow: 0 6px 14px rgba(0,0,0,.35), inset 0 0 0 3px rgba(255,255,255,.45);
    }
    .mini.size-1{ width:30px; height:30px; }
    .mini.size-2{ width:38px; height:38px; }
    .mini.size-3{ width:46px; height:46px; }
    .mini.blue{
      background: radial-gradient(110% 110% at 30% 25%, #5bb6ff 0%, #2585e6 40%, #1e6cc3 72%, #15539a 100%);
      border:2px solid #87c8ff;
    }
    .mini.orange{
      background: radial-gradient(110% 110% at 30% 25%, #ffc078 0%, #ff9f3f 40%, #f27f1a 70%, #c86506 100%);
      border:2px solid #ffd79f;
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
      background: linear-gradient(180deg, rgba(0,0,0,.55), rgba(20,24,34,.65));
      border:1px solid rgba(212,175,55,.55);
      border-radius:999px;
      padding: 2px 6px;
      font-size:12px;
      letter-spacing:.5px;
      box-shadow: 0 2px 8px rgba(0,0,0,.35), 0 0 10px rgba(212,175,55,.25);
      pointer-events:none;
      user-select:none;
      text-shadow: 0 1px 2px rgba(0,0,0,.6);
    }

    .count{ font-size:13px; color:#e2e8f0; }
    .count.zero{ animation: blinkRed .9s infinite; font-weight:800; }

    .left{ grid-area:left; }
    .right{ grid-area:right; }

    /* Footer */
    .footer{
      grid-area:footer; text-align:center; color:#a8b3c5; font-size:12px;
      text-shadow: 0 2px 8px rgba(0,0,0,.45);
    }

    /* Toast message */
    .toast{
      position:fixed; left:50%; bottom:18px; transform: translateX(-50%);
      background: linear-gradient(180deg, #1a243d, #121826);
      color:#fff; padding:10px 14px; border-radius:999px; font-size:13px;
      border:1px solid rgba(212,175,55,.55);
      box-shadow: 0 12px 28px rgba(0,0,0,.45), 0 0 20px rgba(212,175,55,.25);
      opacity:0; pointer-events:none; transition:opacity .18s ease, transform .18s ease;
    }
    .toast.show{ opacity:.95; transform: translateX(-50%) translateY(-4px); }
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
        ：“放置或移動（只可移動自己最上層） • 更大的棋能覆蓋更細棋（<u>包括同色</u>）使其不能移動，除非更大的一方移開 • 勝利：只計每格最上層，先連成一線（橫/直/斜）者勝”
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
    <div class="footer">豪華暗色主題 · 霓虹亮燈 · 大中小清晰辨識 · 無提示 PVP · 動畫：彈跳/壓住 · 同色大覆細允許 · 可隨時改為移動</div>

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

    // --- Elements ---
    const boardEl = document.getElementById("board");
    const toastEl = document.getElementById("toast");
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

    // Tray selection（支持改變主意：再次點同一按鈕可取消；或點棋盤移動）
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
      turnText.textContent = current==="blue" ? "藍" : "橙";

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

      const tp = topPiece(index);

      // 1) 已在托盤選了大小 → 嘗試放置（允許同色大覆細）
      if(selectedSize !== null){
        if(!canPlace(current, selectedSize, index)){
          showToast("呢步唔合法（不可覆同大小或較大）");
          return;
        }
        placePiece(current, selectedSize, index);
        counts[current][selectedSize]--;
        selectedSize = null; clearTrayActive();

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
        showToast("移動唔合法（不可覆同大小或較大）");
        return;
      }

      // 合法移動
      movePiece(current, fromTop.size, selectedFrom, index);
      selectedFrom = null;

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

    // Init
    render();
  })();
  </script>
</body>

