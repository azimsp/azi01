# azi01 1354646;jkpoihiufgtdershk64ju31354365hhkhi543546634652kk1256468jhlihlijljhlhlojkvh,jv3221355jhgjhgohhf,'pk;ojligyfgrdjh;
my NFT01
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Ludo (Mench) – 2D Web Version</title>
<style>
  :root {
    --cell: 38px;
    --gap: 2px;
    --bg: #f4f6f8;
    --grid-border: #dadde1;
    --track: #e9ecef;
    --home: #f8f9fa;
    --safe: #d7f3d1;
  }
  * { box-sizing: border-box; }
  body {
    margin: 0; padding: 16px; background: var(--bg); font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji";
    color: #111827;
  }
  h1 { margin: 0 0 8px; font-size: 22px; }
  .app {
    display: grid; grid-template-columns: 1fr auto; gap: 16px; align-items: start; max-width: 1100px; margin: 0 auto;
  }
  .panel { background: white; border: 1px solid var(--grid-border); border-radius: 12px; padding: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }

  /* Board */
  .board-wrapper { display:flex; gap: 16px; flex-wrap: wrap; align-items: start; }
  #board {
    display: grid; grid-template-columns: repeat(15, var(--cell)); grid-template-rows: repeat(15, var(--cell));
    gap: var(--gap); background: var(--grid-border); padding: var(--gap); border-radius: 12px; position: relative;
  }
  .cell { width: var(--cell); height: var(--cell); background: white; border-radius: 6px; position: relative; display:flex; align-items:center; justify-content:center; }
  .track { background: var(--track); }
  .home-row { background: var(--home); }
  .center { background: #fff; border: 2px dashed #c4cbd3; }
  .safe { outline: 3px solid var(--safe); outline-offset: -3px; }

  /* Pawn chips */
  .pawn { width: calc(var(--cell) - 10px); height: calc(var(--cell) - 10px); border-radius: 50%; display: grid; place-items: center; font-size: 12px; font-weight: 700; color: #111; cursor: pointer; box-shadow: inset 0 -3px 0 rgba(0,0,0,0.15), 0 2px 6px rgba(0,0,0,0.15); border: 2px solid rgba(0,0,0,0.15); }
  .p1 { background: #ef4444; color: #fff; }
  .p2 { background: #10b981; color: #fff; }
  .p3 { background: #3b82f6; color: #fff; }
  .p4 { background: #fbbf24; color: #111; }
  .ghost { opacity: 0.5; pointer-events: none; }

  /* Side info */
  .controls { display:flex; gap: 10px; align-items:center; flex-wrap: wrap; }
  button { border: 0; background: #111827; color: white; padding: 10px 14px; border-radius: 10px; font-weight: 700; cursor: pointer; }
  button:disabled { opacity: .5; cursor: not-allowed; }
  .tag { display:inline-flex; align-items:center; gap:8px; background:#eef2ff; color:#3730a3; padding:6px 10px; border-radius:999px; font-weight:700; }
  .legend { display:grid; grid-template-columns: repeat(2, minmax(140px, 1fr)); gap: 10px; }
  .legend .row { display:flex; align-items:center; gap:8px; }
  .swatch { width: 18px; height: 18px; border-radius: 50%; border: 2px solid rgba(0,0,0,0.15); }
  .yard { display:flex; gap:6px; flex-wrap:wrap; min-height: calc(var(--cell) * 2); padding:8px; background:#f9fafb; border-radius: 10px; border:1px dashed #d1d5db; }

  .log { max-height: 260px; overflow:auto; font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; background:#0b1020; color:#d1e7ff; border-radius:10px; padding:10px; }
  .log p { margin: 0 0 6px; white-space: pre-wrap; }

  .hint { font-size: 13px; color: #4b5563; }
  .sep { height:1px; background:#e5e7eb; margin:10px 0; }
</style>
</head>
<body>
  <div class="app">
    <div class="panel">
      <h1>🎲 Ludo (Mench) – 2D Web Version</h1>
      <div class="controls">
        <button id="btnRoll">Roll Dice</button>
        <span id="stateTag" class="tag">Player 1's turn</span>
        <span id="diceFace" class="tag">Dice: –</span>
        <button id="btnReset" style="background:#7c3aed">Reset</button>
      </div>
      <p class="hint">Click a highlighted pawn to move it. Rolling a 6 grants another roll (max 3 in a row). Landing on opponents sends them home. Enter home row via your start square and reach the center with exact count to finish.</p>
      <div class="board-wrapper">
        <div id="board"></div>
        <div style="flex:1; min-width:260px; display:grid; gap:12px;">
          <div class="panel" style="padding:10px;">
            <strong>Reserves (Home)</strong>
            <div class="sep"></div>
            <div class="legend">
              <div class="row"><span class="swatch p1"></span> Player 1</div>
              <div id="yard1" class="yard"></div>
              <div class="row"><span class="swatch p2"></span> Player 2</div>
              <div id="yard2" class="yard"></div>
              <div class="row"><span class="swatch p3"></span> Player 3</div>
              <div id="yard3" class="yard"></div>
              <div class="row"><span class="swatch p4"></span> Player 4</div>
              <div id="yard4" class="yard"></div>
            </div>
          </div>
          <div class="panel" style="padding:10px;">
            <strong>Moves Log</strong>
            <div class="sep"></div>
            <div id="log" class="log"></div>
          </div>
        </div>
      </div>
    </div>
  </div>

<script>
(function(){
  const N = 15; // 15x15 grid
  const boardEl = document.getElementById('board');
  const btnRoll = document.getElementById('btnRoll');
  const btnReset = document.getElementById('btnReset');
  const tag = document.getElementById('stateTag');
  const diceFace = document.getElementById('diceFace');
  const logEl = document.getElementById('log');
  const yards = [null, document.getElementById('yard1'), document.getElementById('yard2'), document.getElementById('yard3'), document.getElementById('yard4')];

  // Helpers
  const idx = (r,c) => r * N + c;
  const cellByIdx = (i) => cells[i];

  // Build grid cells
  const cells = [];
  for (let r=0;r<N;r++){
    for (let c=0;c<N;c++){
      const d = document.createElement('div');
      d.className = 'cell';
      d.dataset.r = r; d.dataset.c = c; d.dataset.i = idx(r,c);
      boardEl.appendChild(d);
      cells.push(d);
    }
  }

  // Build main track (52 squares = perimeter without corners)
  const path = [];
  // top row (0,1..13)
  for (let c=1;c<=13;c++) path.push(idx(0,c));
  // right col (1..13,14)
  for (let r=1;r<=13;r++) path.push(idx(r,14));
  // bottom row (14,13..1)
  for (let c=13;c>=1;c--) path.push(idx(14,c));
  // left col (13..1,0)
  for (let r=13;r>=1;r--) path.push(idx(r,0));

  // Mark track cells
  path.forEach(i => cells[i].classList.add('track'));

  // Start indices (midpoints of each side)
  const startIdx = [ idx(0,7), idx(7,14), idx(14,7), idx(7,0) ];
  const startOnPathIndex = [ path.indexOf(startIdx[0]), path.indexOf(startIdx[1]), path.indexOf(startIdx[2]), path.indexOf(startIdx[3]) ];

  // Mark safe squares (starting squares)
  startIdx.forEach(i => cells[i].classList.add('safe'));

  // Home rows (6 each; last is the center shared target)
  const center = idx(7,7);
  const homeRows = [
    [ idx(1,7), idx(2,7), idx(3,7), idx(4,7), idx(5,7), center ], // Player 1 (top -> down)
    [ idx(7,13), idx(7,12), idx(7,11), idx(7,10), idx(7,9), center ], // Player 2 (right -> left)
    [ idx(13,7), idx(12,7), idx(11,7), idx(10,7), idx(9,7), center ], // Player 3 (bottom -> up)
    [ idx(7,1), idx(7,2), idx(7,3), idx(7,4), idx(7,5), center ]  // Player 4 (left -> right)
  ];
  homeRows.flat().forEach(i => cells[i].classList.add('home-row'));
  cells[center].classList.add('center');

  // Game state
  const players = [
    { id:0, name:'Player 1', cls:'p1', startPathIndex:startOnPathIndex[0], home:[], pawns:[] },
    { id:1, name:'Player 2', cls:'p2', startPathIndex:startOnPathIndex[1], home:[], pawns:[] },
    { id:2, name:'Player 3', cls:'p3', startPathIndex:startOnPathIndex[2], home:[], pawns:[] },
    { id:3, name:'Player 4', cls:'p4', startPathIndex:startOnPathIndex[3], home:[], pawns:[] },
  ];

  const PAWNS_PER = 4;
  const HOME_LEN = 6; // last is center

  let cur = 0; // current player index 0..3
  let dice = null;
  let canRoll = true;
  let consecutiveSixes = 0;

  function resetGame(){
    // clear yards
    for(let k=1;k<=4;k++) yards[k].innerHTML = '';
    // init players
    players.forEach(p => {
      p.pawns = [];
      for (let i=0;i<PAWNS_PER;i++){
        p.pawns.push({ state:'reserve', // 'reserve' | 'track' | 'home'
                       pos:null, // path index if on track
                       homeIndex:null, // 0..5 if in home row
                       id:i });
      }
    });
    cur = 0; dice = null; canRoll = true; consecutiveSixes = 0;
    updateHeader();
    draw();
    logEl.innerHTML = '';
    log(`${players[cur].name} starts.`);
  }

  function updateHeader(){
    tag.textContent = `${players[cur].name}'s turn`;
    diceFace.textContent = `Dice: ${dice==null?'–':dice}`;
    btnRoll.disabled = !canRoll;
  }

  function log(msg){
    const p = document.createElement('p');
    p.textContent = msg;
    logEl.prepend(p);
  }

  function roll(){
    if(!canRoll) return;
    dice = Math.floor(Math.random()*6)+1;
    if (dice === 6) consecutiveSixes++; else consecutiveSixes = 0;
    updateHeader();
    log(`${players[cur].name} rolled ${dice}.`);

    // If three sixes in a row -> turn forfeited
    if (consecutiveSixes >= 3){
      log(`${players[cur].name} rolled three 6s in a row. Turn lost.`);
      dice = null; consecutiveSixes = 0; canRoll = true; nextTurn(); return;
    }

    // Highlight movable pawns; if none, auto pass (unless 6 allows entry and blocked? entry can capture)
    const moves = getMovablePawns(players[cur], dice);
    if (moves.length === 0){
      log(`No legal move for ${players[cur].name}.`);
      // extra roll only if 6 AND a move was possible? In classic rules you still get another roll on 6 even if no move? We'll allow extra roll on 6 regardless.
      if (dice === 6){ canRoll = true; updateHeader(); return; }
      dice = null; canRoll = true; nextTurn(); return;
    }
    // Wait for pawn click
    canRoll = false; updateHeader();
    highlightMovables(players[cur], moves);
  }

  function nextTurn(){
    clearHighlights();
    cur = (cur + 1) % 4; dice = null; canRoll = true; consecutiveSixes = 0; updateHeader();
  }

  function allFinished(p){
    return p.pawns.every(x => x.state==='home' && x.homeIndex===HOME_LEN-1);
  }

  function checkWin(){
    if (allFinished(players[cur])){
      log(`${players[cur].name} has won the game! 🎉`);
      alert(`${players[cur].name} wins!`);
      canRoll = false; btnRoll.disabled = true;
    }
  }

  function getMovablePawns(p, d){
    const list = [];
    p.pawns.forEach(pawn => {
      if (pawn.state==='reserve'){
        if (d===6){ // entering is always allowed; on entry landing may capture
          list.push({pawn, type:'enter', target:p.startPathIndex});
        }
      } else if (pawn.state==='track'){
        const target = computeTrackAdvance(p, pawn.pos, d);
        if (target.kind==='track' || target.kind==='enter-home' || target.kind==='home'){
          // entering home requires exact steps to center via home row, handled inside
          list.push({pawn, type:'advance', target});
        }
      } else if (pawn.state==='home'){
        const next = pawn.homeIndex + d;
        if (next < HOME_LEN){
          list.push({pawn, type:'home-advance', targetIndex: next});
        }
      }
    });
    return list;
  }

  // Compute where you'd land after d steps from a path index for player p
  // Handles crossing the player's start square into home row.
  function computeTrackAdvance(player, pos, d){
    let i = pos;
    for (let step=0; step<d; step++){
      const next = (i + 1) % path.length;
      // If next is player's start square, stepping off it moves into home row index 0 (only if more steps remain)
      if (next === player.startPathIndex){
        // Move into home row on the following step
        if (step === d-1){
          // exact landing on start square, still on track
          i = next;
        } else {
          // Enter home row
          const remaining = d - step - 1; // steps left inside home row
          if (remaining <= HOME_LEN){
            return { kind:'home', homeIndex: remaining-1 }; // after consuming remaining, final index in home row
          } else {
            return { kind:'blocked' };
          }
        }
      } else {
        i = next;
      }
    }
    return { kind:'track', pathIndex: i };
  }

  function isSafeCell(pathIndex){
    const cell = path[pathIndex];
    return startIdx.includes(cell); // starting squares are safe
  }

  function highlightMovables(player, moves){
    clearHighlights();
    moves.forEach(m => {
      // Create ghost marker in target cell for preview
      if (m.type==='enter'){
        const cell = cells[path[player.startPathIndex]];
        addGhost(cell, player);
      } else if (m.type==='advance'){
        if (m.target.kind==='track'){
          addGhost(cells[path[m.target.pathIndex]], player);
        } else if (m.target.kind==='home'){
          const t = homeRows[player.id][m.target.homeIndex];
          addGhost(cells[t], player);
        } else if (m.target.kind==='enter-home'){
          // not used in current logic
        }
      } else if (m.type==='home-advance'){
        const t = homeRows[player.id][m.targetIndex];
        addGhost(cells[t], player);
      }
    });

    // Enable clicking only on player's pawns that are legal to move
    player.pawns.forEach(pawn => {
      const el = document.querySelector(`.pawn[data-owner='${player.id}'][data-id='${pawn.id}']`);
      if (!el) return;
      const legal = moves.some(m => m.pawn===pawn);
      if (legal){
        el.classList.add('can-move');
        el.addEventListener('click', onPawnClick);
      }
    });
  }

  function clearHighlights(){
    document.querySelectorAll('.ghost').forEach(x=>x.remove());
    document.querySelectorAll('.pawn').forEach(el=>{
      el.classList.remove('can-move');
      el.replaceWith(el.cloneNode(true)); // remove listeners
    });
  }

  function addGhost(cell, player){
    const g = document.createElement('div');
    g.className = `pawn ghost ${player.cls}`;
    cell.appendChild(g);
  }

  function onPawnClick(e){
    const el = e.currentTarget;
    const owner = parseInt(el.dataset.owner,10);
    const pid = parseInt(el.dataset.id,10);
    const p = players[owner];
    const pawn = p.pawns.find(x => x.id===pid);

    const moves = getMovablePawns(p, dice);
    const chosen = moves.find(m => m.pawn===pawn);
    if (!chosen) return;

    // Execute move
    clearHighlights();
    if (chosen.type==='enter'){
      pawn.state='track'; pawn.pos = p.startPathIndex;
      captureAt(p, pawn.pos);
      log(`${p.name} enters a pawn.`);
    } else if (chosen.type==='advance'){
      if (chosen.target.kind==='track'){
        pawn.pos = chosen.target.pathIndex;
        captureAt(p, pawn.pos);
        log(`${p.name} moves on track to ${pawn.pos+1}/52${isSafeCell(pawn.pos)?' (safe)':''}.`);
      } else if (chosen.target.kind==='home'){
        pawn.state='home'; pawn.homeIndex = chosen.target.homeIndex;
        log(`${p.name} enters home row at step ${pawn.homeIndex+1}/${HOME_LEN}.`);
      }
    } else if (chosen.type==='home-advance'){
      pawn.homeIndex = chosen.targetIndex;
      log(`${p.name} advances in home row to ${pawn.homeIndex+1}/${HOME_LEN}.`);
    }

    draw();

    // Check finish for this pawn
    if (pawn.state==='home' && pawn.homeIndex===HOME_LEN-1){
      log(`${p.name} finished a pawn! ✨`);
      checkWin();
    }

    // Handle extra roll
    if (dice === 6){
      canRoll = true; updateHeader();
    } else {
      dice = null; canRoll = true; nextTurn();
    }
  }

  function captureAt(player, pathIndex){
    const cell = path[pathIndex];
    // gather all pawns on this cell from other players
    players.forEach(op => {
      if (op===player) return;
      op.pawns.forEach(pn => {
        if (pn.state==='track' && path[pn.pos]===cell){
          // If landing cell is safe, no capture
          if (isSafeCell(pathIndex)) return;
          // send home
          pn.state='reserve'; pn.pos=null; pn.homeIndex=null;
          log(`${player.name} captures a pawn of ${op.name}.`);
        }
      });
    });
  }

  function draw(){
    // clear all pawns
    document.querySelectorAll('.pawn').forEach(x=>x.remove());
    // draw reserves
    players.forEach((p,pi)=>{
      const yard = yards[pi+1];
      yard.innerHTML = '';
      p.pawns.filter(x=>x.state==='reserve').forEach(x=>{
        const el = makePawnEl(p, x);
        yard.appendChild(el);
      });
    });

    // draw board pawns
    players.forEach(p=>{
      p.pawns.forEach(pawn=>{
        if (pawn.state==='track'){
          const cell = cells[path[pawn.pos]];
          const el = makePawnEl(p, pawn);
          cell.appendChild(el);
        } else if (pawn.state==='home'){
          const cell = cells[homeRows[p.id][pawn.homeIndex]];
          const el = makePawnEl(p, pawn);
          cell.appendChild(el);
        }
      });
    });
  }

  function makePawnEl(player, pawn){
    const el = document.createElement('div');
    el.className = `pawn ${player.cls}`;
    el.dataset.owner = player.id;
    el.dataset.id = pawn.id;
    el.title = `${player.name} • Pawn ${pawn.id+1}`;
    el.textContent = pawn.id+1;
    return el;
  }

  // Wire buttons
  btnRoll.addEventListener('click', ()=>{
    roll();
  });
  btnReset.addEventListener('click', ()=>{
    resetGame();
  });

  // Init
  resetGame();
})();
</script>
</body>
</html>


knlkhlkkjind/J?PIj{KWSp;
kLHDHEI?BFKOWPFk:OWO{W{?J:W?OK13546LWK:OKsk

zfdxtrdytfvjhbkjlm;,.;aswxdcfvgbhnjmk,l.;
JIJDI:HROIH?JSOEKD;
G>IEHIJW?OJPEJF?DWJG?IJHG/HIh/whg/hw
.HUKDh>HEUH.?HI>?JEG?ijG?IJ?POWI"PI?PKP_WQO[aq_LPO_QPLOQ}_O_'k;d;lkfos
uguylurre4w3qeawdzsxcfvghbjnmkl,.;
