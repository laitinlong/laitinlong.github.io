
<script>
(function(){
const sizeNames={1:"小",2:"中",3:"大"},winLines=[[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]],WIN_DELAY=5000,CONFIRM_DELAY=2000;
const boardEl=document.getElementById("board"),turnDot=document.getElementById("turnDot"),turnText=document.getElementById("turnText");
const restartBtn=document.getElementById("restartBtn"),swapBtn=document.getElementById("swapBtn"),modeBtn=document.getElementById("modeBtn");
const arrowLayer=document.getElementById('arrowLayer'),arrowPath=document.getElementById('arrowPath'),msgEl=document.getElementById('msg');
const trayBlue=document.getElementById('trayBlue');

let board,counts,current,selectedSize,gameOver;
let teachingMode=true,stepIndex=0,movingFromIndex=null,pvpSelectedFrom=null;
let winLetters={},currentArrow=null,ghostAnim=null,winPulse=new Set(),winLineIdx=null;

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

function resetCommon(){ board=Array.from({length:9},()=>[]); counts={blue:{1:2,2:2,3:2},orange:{1:2,2:2,3:2}}; selectedSize=null; gameOver=false; movingFromIndex=null; pvpSelectedFrom=null; currentArrow=null; clearArrow(); winPulse.clear(); winLineIdx=null; render(); clearHints(); clearTrayGlow(); }
function clearWinLettersDOM(){ Array.from(boardEl.children).forEach(c=>{ const ov=c.querySelector('.cell-overlay'); if(ov) ov.innerHTML=""; }); winLetters={}; }

function resetTeaching(){ clearWinLettersDOM(); teachingMode=true; stepIndex=0; modeBtn.textContent="退出教學模式"; restartBtn.style.display="none"; swapBtn.style.display="none"; resetCommon(); current="blue"; render(); showNextHint(); }
function resetPVP(start="blue"){ clearWinLettersDOM(); teachingMode=false; modeBtn.textContent="開始教學模式"; restartBtn.style.display=""; swapBtn.style.display=""; resetCommon(); current=start; render(); }

restartBtn.addEventListener("click",()=>{ if(gameOver) return; if(!teachingMode) resetPVP("blue"); });
swapBtn.addEventListener("click",()=>{ if(gameOver) return; if(!teachingMode){ current=(current==="blue")?"orange":"blue"; resetPVP(current); }});
modeBtn.addEventListener("click",()=>{ teachingMode?resetPVP("blue"):resetTeaching(); });

document.querySelectorAll(".tray-btn").forEach(btn=>{
  btn.addEventListener("click",()=>{
    if(teachingMode) return;
    if(gameOver) return;
    const player=btn.dataset.player,size=Number(btn.dataset.size);
    if(player!==current) return;
    if(counts[player][size]<=0) return;
    if(pvpSelectedFrom!==null){ const el=boardEl.children[pvpSelectedFrom]; el&&el.classList.remove('source-cue'); pvpSelectedFrom=null; }
    if(selectedSize===size){ selectedSize=null; clearTrayGlow(); return; }
    selectedSize=size; clearTrayGlow(); btn.classList.add("glow-green","active");
  });
});

function layoutArrowLayer(){
  const vv=window.visualViewport;
  if(vv){ arrowLayer.style.width=vv.width+"px"; arrowLayer.style.height=vv.height+"px"; arrowLayer.style.left="0px"; arrowLayer.style.top="0px"; arrowLayer.setAttribute('viewBox',`0 0 ${vv.width} ${vv.height}`);}
  else{ arrowLayer.style.width="100vw"; arrowLayer.style.height="100vh"; arrowLayer.style.left="0px"; arrowLayer.style.top="0px"; arrowLayer.setAttribute('viewBox',`0 0 ${window.innerWidth} ${window.innerHeight}`);}
}
function getCenter(el){ if(!el) return null; const r=el.getBoundingClientRect(); return {x:r.left+r.width/2,y:r.top+r.height/2,w:r.width,h:r.height}; }
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
  arrowPath.setAttribute('stroke',getComputedStyle(document.documentElement).getPropertyValue(kind==='place'?'--arrowPlace':'--arrowMove'));
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
function canMove(player,size,from,to){if(from===to)return false;const ft=topPiece(from);if(!ft||ft.player!==player||ft.size!==size)return false;const tt=topPiece(to);return !tt||size>tt.size;}
function checkWin(p){return winLines.some(line=>line.every(i=>{const t=topPiece(i);return t&&t.player===p;}));}
function getWinningLine(p){for(const l of winLines){if(l.every(i=>{const t=topPiece(i);return t&&t.player===p;})) return l}return null}
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
  })
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

function onCellClick(){
  if(gameOver) return;
  if(teachingMode){
    const mv=SCRIPT[stepIndex]; if(!mv||mv.actor!=='blue') return;
    if(uiLocked) return;
    clearHints(); clearArrow();

    if(mv.type==='place'){
      const dst=boardEl.children[mv.to];
      lock();
      setTimeout(()=>{
        const trayBtn=document.querySelector(`#trayBlue .tray-btn[data-size="${mv.size}"]`);
        const dot=trayBtn?trayBtn.querySelector('.mini'):trayBtn; const dstEl=boardEl.children[mv.to];
        ghostMove(dot,dstEl,'blue',mv.size,600).then(()=>{
          board[mv.to].push({player:'blue',size:mv.size});
          counts.blue[mv.size]--; stepIndex++; clearHints(); clearTrayGlow(); clearArrow();
          if(checkWin('blue')){ startWinSequence(); unlock(); return; }
          current='orange'; render(); setTimeout(runAIMoveIfAny,450);
        });
      },CONFIRM_DELAY);
    }else{
      const src=boardEl.children[mv.from],dst=boardEl.children[mv.to];
      const pos=getCenter(src);
      lock();
      setTimeout(()=>{
        board[mv.from].pop(); render();
        ghostMove({x:pos.x,y:pos.y},dst,'blue',mv.size,650).then(()=>{
          board[mv.to].push({player:'blue',size:mv.size});
          stepIndex++; movingFromIndex=null; clearHints(); clearArrow();
          if(checkWin('blue')){ startWinSequence(); unlock(); return; }
          current='orange'; render(); setTimeout(runAIMoveIfAny,450);
        });
      },CONFIRM_DELAY);
    }
    return;
  }
  handlePVP(...arguments);
}

function runAIMoveIfAny(){
  if(gameOver||stepIndex>=SCRIPT.length){showNextHint();unlock();return}
  const mv=SCRIPT[stepIndex]; if(mv.actor!=='orange'){showNextHint();unlock();return}
  showNextHint(true);

  if(mv.type==='place'){
    const trayBtn=[...document.querySelectorAll('#trayOrange .tray-btn')].find(b=>Number(b.dataset.size)===mv.size);
    const dot=trayBtn?trayBtn.querySelector('.mini'):trayBtn; const dst=boardEl.children[mv.to];
    ghostMove(dot,dst,'orange',mv.size,600).then(()=>{
      board[mv.to].push({player:'orange',size:mv.size});
      counts.orange[mv.size]--; stepIndex++; clearTrayGlow(); clearHints(); clearArrow();
      if(checkWin('orange')){ gameOver=true; render(); alert("橙方勝"); unlock(); return; }
      current='blue'; render(); showNextHint(); unlock();
    });
  }else{
    const src=boardEl.children[mv.from],dst=boardEl.children[mv.to],pos=getCenter(src);
    board[mv.from].pop(); render();
    ghostMove({x:pos.x,y:pos.y},dst,'orange',mv.size,650).then(()=>{
      board[mv.to].push({player:'orange',size:mv.size});
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
    if(!canPlace(current,selectedSize,index)) return;
    board[index].push({player:current,size:selectedSize}); counts[current][selectedSize]--; selectedSize=null; clearTrayGlow(); render();
    if(checkWin(current)){ alert((current==='blue'?'綠':'橙')+'方勝'); gameOver=true; return; }
    switchTurn(); return;
  }
  if(pvpSelectedFrom===null){
    if(!tp) return;
    if(tp.player!==current) return;
    pvpSelectedFrom=index; boardEl.children[index].classList.add('source-cue'); return;
  }
  if(index===pvpSelectedFrom){ boardEl.children[index].classList.remove('source-cue'); pvpSelectedFrom=null; return; }
  const fromTop=topPiece(pvpSelectedFrom);
  if(!fromTop){ pvpSelectedFrom=null; clearHints(); return; }
  if(!canMove(current,fromTop.size,pvpSelectedFrom,index)) return;
  board[index].push(board[pvpSelectedFrom].pop()); clearHints(); pvpSelectedFrom=null; render();
  if(checkWin(current)){ alert((current==='blue'?'綠':'橙')+'方勝'); gameOver=true; return; }
  switchTurn();
}

let uiLocked=false;
function lock(){ uiLocked=true; document.body.style.pointerEvents='none'; }
function unlock(){ uiLocked=false; document.body.style.pointerEvents='auto'; }

function hint(t){ /* 已移除，不顯示任何文字 */ }

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
layoutArrowLayer(); resetTeaching();
})();
</script>
