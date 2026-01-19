

<html lang="zh-Hant">

  <title>超級過三關</title>
  <style>
    :root{
      --blue:#1e90ff;
      --orange:#ff8c00;
      --board-bg:#f7f7f9;
      --text:#222;
      --cell-size: min(22vmin, 130px);
      --gap: 10px;
      --hint:#4caf50;
      --move:#ff6f00;
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

    /* Header */
    .header{ grid-area:header; display:flex; flex-direction:column; align-items:center; gap:8px; }
    .title{
      margin:0; font-weight:900; letter-spacing:.5px;
      color: var(--blue);
      font-size: clamp(22px, 5vw, 40px);
      text-shadow: 0 2px 10px rgba(30,144,255,.15);
    }
    .header-bar{ display:flex; align-items:center; gap:10px; }
    .dot{ width:14px; height:14px; border-radius:50%; box-shadow: inset 0 0 0 2px rgba(255,255,255,.6); }
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
    .turn-text{ font-weight:800; font-size:14px; color:#333; }

    .btn{
      border:1px solid #ccc; background:#fff; color:#222;
      padding:8px 12px; border-radius:10px; cursor:pointer;
      display:inline-flex; align-items:center; justify-content:center;
      font-size:14px; transition:.15s; user-select:none;
    }
    .btn:hover{ transform: translateY(-1px); box-shadow:0 3px 10px rgba(0,0,0,.08); }
    .btn:active{ transform: translateY(1px); }

    /* Trays */
    .tray{
      grid-area:left;
      background:#fff; border:1px solid #e6e6e6; border-radius:14px;
      padding:12px; box-shadow: 0 6px 16px rgba(0,0,0,.06);
    }
    .right{ grid-area:right; }
    .tray-grid{ display:grid; grid-template-columns: repeat(3, 1fr); gap:10px; }
    .tray-btn{
      display:flex; align-items:center; justify-content:center;
      padding:8px; cursor:pointer; border-radius:12px;
      border:1px solid #ddd; background:#fafafa; transition:.15s; min-height:92px;
    }
    .tray-btn:hover{ background:#f5f5f5; transform: translateY(-1px); }
    .tray-btn.active{ border-color:#888; box-shadow:0 4px 12px rgba(0,0,0,.08); background:#fff; }

    .mini{
      position: relative; border-radius:50%;
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
    .mini.blue.size-1{ border-width:2px; } .mini.blue.size-2{ border-width:4px; } .mini.blue.size-3{ border-width:6px; }
    .mini.orange.size-1{ border-width:2px; } .mini.orange.size-2{ border-width:4px; } .mini.orange.size-3{ border-width:6px; }

    /* Board */
    .board-wrap{ grid-area:board; display:flex; justify-content:center; align-items:center; }
    .board{
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

    /* Place hint（綠） */
    .cell.hint{
      box-shadow: inset 0 0 0 3px var(--hint);
      animation: pulseHint 1.2s ease-in-out infinite;
    }
    @keyframes pulseHint{
      0%{ box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 0 rgba(76,175,80,.35); }
      50%{ box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 8px rgba(76,175,80,.0); }
      100%{ box-shadow: inset 0 0 0 3px var(--hint), 0 0 0 0 rgba(76,175,80,.35); }
    }

    /* Move target（橙） */
    .cell.hint-move{
      box-shadow: inset 0 0 0 3px var(--move), 0 0 0 6px rgba(255,111,0,.22);
      animation: targetPulse 1.05s ease-in-out infinite;
    }
    @keyframes targetPulse{ 0%{ transform:scale(1) } 50%{ transform:scale(1.02) } 100%{ transform:scale(1) } }

    /* Move source（藍環） */
    .cell.source-cue{
      box-shadow: inset 0 0 0 3px #64b5f6, 0 0 0 6px rgba(100,181,246,.18);
    }

    /* Piece */
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

    /* 中央尺寸標籤：大／中／小 */
    .size-badge{
      position:absolute;
      left:50%; top:50%; transform: translate(-50%,-50%);
      color:#fff; font-weight:900; letter-spacing:.5px;
      background: rgba(0,0,0,.35);
      border-radius:999px; padding: 2px 8px;
      box-shadow: 0 2px 6px rgba(0,0,0,.25);
      user-select:none; pointer-events:none;
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
  </style>
</head>
<body>
  <div class="app">
    <!-- Header -->
    <div class="header" aria-label="controls">
      <h1 class="title">超級過三關</h1>
      <div class="header-bar">
        <span id="turnDot" class="dot blue" aria-hidden="true"></span>
        <span id="turnText" class="turn-text">輪到：藍</span>
        <button id="restartScriptBtn" class="btn" aria-label="重新開始">重新開始</button>
        <button id="exitScriptBtn" class="btn" aria-label="退出劇本">退出劇本</button>
      </div>
    </div>

    <!-- Left Tray (Blue) -->
    <div class="tray" id="trayBlue" aria-label="藍方托盤">
      <div class="tray-grid">
        <div class="tray-btn" data-player="blue" data-size="3"><div class="mini blue size-3"></div></div>
        <div class="tray-btn" data-player="blue" data-size="2"><div class="mini blue size-2"></div></div>
        <div class="tray-btn" data-player="blue" data-size="1"><div class="mini blue size-1"></div></div>
      </div>
    </div>

    <!-- Board -->
    <div class="board-wrap">
      <div id="board" class="board" aria-label="3x3 棋盤"></div>
    </div>

    <!-- Right Tray (Orange / AI) -->
    <div class="tray right" id="trayOrange" aria-label="橙方托盤">
      <div class="tray-grid">
        <div class="tray-btn" data-player="orange" data-size="3"><div class="mini orange size-3"></div></div>
        <div class="tray-btn" data-player="orange" data-size="2"><div class="mini orange size-2"></div></div>
        <div class="tray-btn" data-player="orange" data-size="1"><div class="mini orange size-1"></div></div>
      </div>
    </div>
  </div>

  <script>
  (function(){
    // --- State & Rules ---
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
    let gameOver = false;

    // --- Script: 13 固定步驟（你提供）---
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
    let scriptedMode = true;

    // --- Elements ---
    const boardEl = document.getElementById("board");
    const turnDot = document.getElementById("turnDot");
    const turnText = document.getElementById("turnText");
    const restartScriptBtn = document.getElementById("restartScriptBtn");
    const exitScriptBtn = document.getElementById("exitScriptBtn");

    // Build 9 cells
    for(let i=0;i<9;i++){
      const c = document.createElement("div");
      c.className = "cell";
      c.dataset.index = i;
      c.addEventListener("click", ()=>onCellClick(i));
      boardEl.appendChild(c);
    }

    // Tray (玩家可點，但劇本會自動預選大小)
    document.querySelectorAll(".tray-btn").forEach(btn=>{
      btn.addEventListener("click", ()=>{
        if(gameOver || !scriptedMode) return;
        const mv = SCRIPT[stepIndex];
        if(!mv || mv.actor!=='blue' || mv.type!=='place') return;
        const size = Number(btn.dataset.size);
        if(size !== mv.size) return;
        if(counts.blue[size] <= 0) return;
        selectedSize = size;
        document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("active"));
        btn.classList.add("active");
      });
    });

    // Controls
    restartScriptBtn.addEventListener("click", ()=>restartScript());
    exitScriptBtn.addEventListener("click", ()=>{
      scriptedMode = false;
      clearHints();
    });

    // --- Interaction ---
    function onCellClick(index){
      if(gameOver) return;

      if(!scriptedMode){
        handleFreePlay(index);
        return;
      }

      const mv = SCRIPT[stepIndex];
      if(!mv) return;

      if(mv.actor==='blue'){
        if(mv.type==='place'){
          // 劇本預選大小，點提示格即可
          if(selectedSize !== mv.size) return;
          if(index !== mv.to) return;
          if(!canPlace('blue', mv.size, index)) return;

          placePiece('blue', mv.size, index);
          counts.blue[mv.size]--;
          selectedSize = null; clearTrayActive();
          stepIndex++;

          if(checkWin('blue')){ gameOver=true; render(); setTimeout(()=>alert("藍方勝"),10); return; }
          switchTurn();
          setTimeout(()=>runAIMoveIfAny(), 600);
          return;
        }else{
          // 移動步：單擊目標格
          if(index !== mv.to) return;
          const top = topPiece(mv.from);
          if(!top || top.player!=='blue' || top.size!==mv.size) return;
          if(!canMove('blue', mv.size, mv.from, mv.to)) return;

          movePiece('blue', mv.size, mv.from, mv.to);
          stepIndex++;

          if(checkWin('blue')){ gameOver=true; render(); setTimeout(()=>alert("藍方勝"),10); return; }
          switchTurn();
          setTimeout(()=>runAIMoveIfAny(), 600);
          return;
        }
      }
    }

    function runAIMoveIfAny(){
      if(gameOver || stepIndex >= SCRIPT.length) { showNextHint(); return; }
      const mv = SCRIPT[stepIndex];
      if(mv.actor !== 'orange'){ showNextHint(); return; }

      current = 'orange'; render();

      if(mv.type === 'place'){
        if(!canPlace('orange', mv.size, mv.to)) return;
        placePiece('orange', mv.size, mv.to);
        counts.orange[mv.size]--;
      }else{
        const top = topPiece(mv.from);
        if(!top || top.player!=='orange' || top.size!==mv.size || !canMove('orange', mv.size, mv.from, mv.to)) return;
        movePiece('orange', mv.size, mv.from, mv.to);
      }

      if(checkWin('orange')){ gameOver=true; render(); setTimeout(()=>alert("橙方勝"),10); return; }
      stepIndex++;
      switchTurn();
      showNextHint();
    }

    // --- Free PVP (after exit) ---
    let selectedFrom = null;
    function handleFreePlay(index){
      const tp = topPiece(index);
      if(selectedSize !== null){
        if(!canPlace(current, selectedSize, index)) return;
        placePiece(current, selectedSize, index);
        counts[current][selectedSize]--;
        selectedSize = null; clearTrayActive();
        if(checkWin(current)){ gameOver=true; render(); setTimeout(()=>alert(current==='blue'?"藍方勝":"橙方勝"),10); return; }
        switchTurn();
        return;
      }
      if(selectedFrom === null){
        if(!tp || tp.player !== current) return;
        selectedFrom = index; render();
        return;
      }
      const fromTop = topPiece(selectedFrom);
      if(!fromTop || fromTop.player !== current){ selectedFrom=null; render(); return; }
      if(selectedFrom === index){ selectedFrom=null; render(); return; }
      if(!canMove(current, fromTop.size, selectedFrom, index)) return;

      movePiece(current, fromTop.size, selectedFrom, index);
      selectedFrom = null;
      if(checkWin(current)){ gameOver=true; render(); setTimeout(()=>alert(current==='blue'?"藍方勝":"橙方勝"),10); return; }
      switchTurn();
    }

    // --- Rules & Rendering ---
    function render(){
      // Turn indicator
      turnDot.className = "dot " + (current==="blue"?"blue":"orange");
      document.getElementById("turnText").textContent = "輪到：" + (current==="blue"?"藍":"橙");

      // Board pieces
      for(let i=0;i<9;i++){
        const cellEl = boardEl.children[i];
        const old = cellEl.querySelector(".piece");
        if(old) old.remove();

        const top = topPiece(i);
        if(top){
          const p = document.createElement("div");
          p.className = `piece ${top.player==='blue'?'blue-piece':'orange-piece'} size-${top.size}`;

          // 尺寸標籤（大/中/小）
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
    }

    function clearTrayActive(){
      document.querySelectorAll(".tray-btn").forEach(b=>b.classList.remove("active"));
    }
    function clearHints(){
      Array.from(boardEl.children).forEach(c=>c.classList.remove("hint","hint-move","source-cue"));
    }

    function canPlace(player,size,index){
      const stack = board[index];
      const top = stack.length ? stack[stack.length-1] : null;
      if(!top) return true;
      return size > top.size; // 大吃小
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
      selectedSize = null; gameOver = false;
      stepIndex = 0; scriptedMode = true;
      clearTrayActive(); clearHints();
      render();
      showNextHint();
    }

    // 提示：落子 = 目標格綠光；移動 = 目標格橙光 + 來源藍環；自動預選大小
    function showNextHint(){
      clearHints();
      if(!scriptedMode || gameOver || stepIndex >= SCRIPT.length) return;
      const mv = SCRIPT[stepIndex];
      if(mv.actor==='blue'){
        if(mv.type==='place'){
          highlightTray('blue', mv.size);
          selectedSize = mv.size; // 預選
          const cell = boardEl.children[mv.to]; if(cell) cell.classList.add("hint");
        }else{
          const src = boardEl.children[mv.from]; if(src) src.classList.add("source-cue");
          const dst = boardEl.children[mv.to];   if(dst) dst.classList.add("hint-move");
        }
      }
    }
    function highlightTray(player,size){
      document.querySelectorAll(".tray-btn").forEach(b=>{
        b.classList.remove("active");
        if(b.dataset.player===player && Number(b.dataset.size)===size) b.classList.add("active");
      });
    }

    // Init
    render();
    showNextHint();
  })();
  </script>
</body>
</html>
