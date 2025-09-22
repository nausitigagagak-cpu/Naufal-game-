# Naufal-game 
<!doctype html>

<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no" />
  <title>Tetris Nopal Asli</title>
  <style>
    :root{--bg:#0b1020;--cell:#0d1226;--panel:#0f1724}
    html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial}
    body{background:linear-gradient(180deg,#071029 0%, #071827 100%);color:#e6eef8;display:flex;align-items:center;justify-content:center;padding:12px}
    .wrap{width:100%;max-width:960px;display:grid;grid-template-columns:1fr 300px;gap:16px}
    .board-panel{background:var(--panel);padding:12px;border-radius:12px;box-shadow:0 8px 30px rgba(2,6,23,.6)}
    canvas{display:block;background:var(--bg);border-radius:8px;width:100%;height:auto}
    .right{display:flex;flex-direction:column;gap:12px}
    h1{margin:0;font-size:20px}
    .info{display:flex;gap:8px;flex-direction:column}
    .stat{background:rgba(255,255,255,0.02);padding:8px;border-radius:8px}
    .controls{display:flex;gap:8px;flex-wrap:wrap}
    button{background:linear-gradient(180deg,#1e293b,#0b1220);border:1px solid rgba(255,255,255,0.04);color:#dbeafe;padding:10px;border-radius:8px;font-weight:600}
    .touch-controls{display:none;position:fixed;left:0;right:0;bottom:12px;gap:8px;justify-content:center;padding:6px}
    .touch-row{display:flex;gap:8px}
    .touch-btn{width:64px;height:64px;border-radius:12px;font-size:18px;display:flex;align-items:center;justify-content:center;user-select:none}
    .small{font-size:13px;color:#a8b3c9}
    footer{margin-top:8px;font-size:12px;color:#98a3bf}
    @media (max-width:820px){
      .wrap{grid-template-columns:1fr}
      .right{order:2}
      .touch-controls{display:flex}
      .board-panel{padding-bottom:92px}
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="board-panel">
      <h1>Tetris Nopal Asli</h1>
      <p class="small">Main lewat HP — sentuh tombol di bawah untuk kontrol. Highscore tersimpan di browser.</p>
      <canvas id="tetris" width="240" height="480"></canvas>
      <footer>© Tetris Nopal Asli</footer>
    </div><div class="right">
  <div class="info stat">
    <div>Score: <span id="score">0</span></div>
    <div>Lines: <span id="lines">0</span></div>
    <div>Level: <span id="level">1</span></div>
    <div>Highscore: <span id="high">0</span></div>
  </div>
  <div class="stat">
    <div class="controls">
      <button id="start">Start / Restart</button>
      <button id="pause">Pause</button>
      <button id="sound">Toggle Sound</button>
    </div>
  </div>
  <div class="stat">
    <div class="small">Kontrol fisik (keyboard): ← → untuk gerak, ↑ rotasi, ↓ soft drop, Spasi hard drop</div>
    <div class="small" style="margin-top:6px">Versi ini dibuat untuk dimainkan langsung dari browser HP (tanpa server). Jika mau hosting, lihat panduan di bawah dokumen.</div>
  </div>
</div>

  </div>  <div class="touch-controls" id="touchControls">
    <div class="touch-row">
      <div class="touch-btn" id="left">◀</div>
      <div class="touch-btn" id="rotate">⟳</div>
      <div class="touch-btn" id="right">▶</div>
    </div>
    <div class="touch-row">
      <div class="touch-btn" id="drop">↓</div>
      <div class="touch-btn" id="hard">⤓</div>
    </div>
  </div>  <script>
  // ---- BASIC TETRIS ENGINE (mobile-friendly) ----
  const canvas = document.getElementById('tetris');
  const ctx = canvas.getContext('2d');
  const COLS = 10, ROWS = 20;
  const BLOCK = 24; // base pixel size — canvas set to 240x480 -> 10x20
  canvas.width = COLS * BLOCK; canvas.height = ROWS * BLOCK;

  // Colors for tetrominoes
  const colors = [null,'#00f0f0','#0000f0','#f0a000','#f0f000','#00f000','#a000f0','#f00000'];

  // Tetromino definitions (matrix forms)
  const tetrominoes = {
    I: [[1,1,1,1]],
    J: [[2,0,0],[2,2,2]],
    L: [[0,0,3],[3,3,3]],
    O: [[4,4],[4,4]],
    S: [[0,5,5],[5,5,0]],
    T: [[0,6,0],[6,6,6]],
    Z: [[7,7,0],[0,7,7]]
  };
  const pieces = Object.keys(tetrominoes);

  function createMatrix(w,h){
    const m = [];
    while(h--) m.push(new Array(w).fill(0));
    return m;
  }

  function drawCell(x,y,value){
    ctx.fillStyle = value ? colors[value] : '#071029';
    ctx.fillRect(x*BLOCK, y*BLOCK, BLOCK-1, BLOCK-1);
  }

  function drawMatrix(matrix, offset){
    matrix.forEach((row,y) => row.forEach((value,x) => { if(value){ drawCell(x+offset.x,y+offset.y,value); }}));
  }

  const arena = createMatrix(COLS, ROWS);

  function collide(arena, player){
    const [m, o] = [player.matrix, player.pos];
    for(let y=0;y<m.length;y++) for(let x=0;x<m[y].length;x++) if(m[y][x]){
      if(!arena[y+o.y] || arena[y+o.y][x+o.x] !== 0) return true;
    }
    return false;
  }

  function merge(arena, player){
    player.matrix.forEach((row,y)=>row.forEach((value,x)=>{ if(value){ arena[y+player.pos.y][x+player.pos.x]=value; }}));
  }

  function rotate(matrix, dir){
    for(let y=0;y<matrix.length;y++) for(let x=0;x<y;x++){
      [matrix[x][y], matrix[y][x]] = [matrix[y][x], matrix[x][y]];
    }
    if(dir>0) matrix.forEach(row=>row.reverse()); else matrix.reverse();
    return matrix;
  }

  function sweep(){
    let rowCount=0;
    outer: for(let y=arena.length-1;y>=0;--y){
      for(let x=0;x<arena[y].length;x++){ if(arena[y][x]===0) continue outer; }
      const row = arena.splice(y,1)[0].fill(0);
      arena.unshift(row);
      y++;
      rowCount++;
    }
    if(rowCount>0){
      player.score += (100 * Math.pow(2,rowCount-1)) * (player.level);
      player.lines += rowCount;
      player.level = 1 + Math.floor(player.lines / 10);
      updateUI();
    }
  }

  function createPiece(type){
    const m = tetrominoes[type].map(r=>r.slice());
    return m;
  }

  const player = {
    pos: {x:0,y:0},
    matrix: null,
    score:0,lines:0,level:1
  };

  function playerReset(){
    const type = pieces[Math.floor(Math.random()*pieces.length)];
    player.matrix = createPiece(type);
    player.pos.y = 0; player.pos.x = Math.floor((COLS - player.matrix[0].length)/2);
    if(collide(arena, player)){
      arena.forEach(row=>row.fill(0));
      player.score = 0; player.lines = 0; player.level = 1; updateUI();
    }
  }

  function playerMove(dir){ player.pos.x += dir; if(collide(arena, player)) player.pos.x -= dir; }

  function playerDrop(){ player.pos.y++; if(collide(arena, player)){ player.pos.y--; merge(arena, player); sweep(); playerReset(); dropCounter = 0; }}

  function hardDrop(){ while(!collide(arena, player)) player.pos.y++; player.pos.y--; merge(arena, player); sweep(); playerReset(); dropCounter = 0; }

  function playerRotate(dir){
    rotate(player.matrix, dir);
    let offset = 1;
    while(collide(arena, player)){
      player.pos.x += offset;
      offset = -(offset + (offset>0?1:-1));
      if(offset > player.matrix[0].length){ rotate(player.matrix, -dir); break; }
    }
  }

  // UI and Game loop
  let dropCounter = 0; let dropInterval = 1000;
  let lastTime = 0; let running = false; let soundOn = true;

  function update(time=0){
    const delta = time - lastTime; lastTime = time;
    if(running){
      dropCounter += delta;
      dropInterval = Math.max(100, 1000 - (player.level-1)*100);
      if(dropCounter > dropInterval){ playerDrop(); dropCounter = 0; }
    }
    draw();
    requestAnimationFrame(update);
  }

  function draw(){
    ctx.fillStyle = '#071029'; ctx.fillRect(0,0,canvas.width,canvas.height);
    // draw arena
    for(let y=0;y<arena.length;y++) for(let x=0;x<arena[y].length;x++) drawCell(x,y,arena[y][x]);
    // draw player piece
    drawMatrix(player.matrix, player.pos);
  }

  function updateUI(){
    document.getElementById('score').innerText = player.score;
    document.getElementById('lines').innerText = player.lines;
    document.getElementById('level').innerText = player.level;
    const high = Number(localStorage.getItem('tetris_nopal_high')||0);
    if(player.score>high){ localStorage.setItem('tetris_nopal_high', player.score); }
    document.getElementById('high').innerText = Math.max(player.score, high);
  }

  // Input handlers
  document.addEventListener('keydown', e=>{
    if(e.key === 'ArrowLeft') playerMove(-1);
    if(e.key === 'ArrowRight') playerMove(1);
    if(e.key === 'ArrowUp') playerRotate(1);
    if(e.key === 'ArrowDown') playerDrop();
    if(e.code === 'Space') hardDrop();
  });

  // Buttons
  document.getElementById('start').addEventListener('click', ()=>{ running = true; player.score = 0; player.lines = 0; player.level = 1; arena.forEach(r=>r.fill(0)); playerReset(); updateUI(); });
  document.getElementById('pause').addEventListener('click', ()=>{ running = !running; document.getElementById('pause').innerText = running? 'Pause' : 'Resume'; });
  document.getElementById('sound').addEventListener('click', ()=>{ soundOn = !soundOn; document.getElementById('sound').innerText = soundOn? 'Toggle Sound' : 'Sound Off'; });

  // Mobile touch controls
  function bindTouch(id, fn){
    const el = document.getElementById(id);
    let timer;
    el.addEventListener('touchstart', e=>{ e.preventDefault(); fn('start'); if(id==='drop'){ timer = setInterval(()=>fn('repeat'),100); } }, {passive:false});
    el.addEventListener('touchend', e=>{ e.preventDefault(); if(timer) clearInterval(timer); fn('end'); }, {passive:false});
  }
  bindTouch('left', ()=>playerMove(-1));
  bindTouch('right', ()=>playerMove(1));
  bindTouch('rotate', ()=>playerRotate(1));
  bindTouch('drop', ()=>playerDrop());
  bindTouch('hard', ()=>hardDrop());

  // Initialize
  playerReset(); updateUI(); update();

  // Simple sound feedback (optional)
  function beep(freq=440,dur=60){ if(!soundOn) return; try{ const ctxA = new (window.AudioContext||window.webkitAudioContext)(); const o = ctxA.createOscillator(); const g = ctxA.createGain(); o.frequency.value = freq; g.gain.value = 0.02; o.connect(g); g.connect(ctxA.destination); o.start(); setTimeout(()=>{ o.stop(); ctxA.close(); }, dur); }catch(e){}
  }

  // Play beep on merge and line clear
  const originalMerge = merge;
  merge = function(arenaArg, playerArg){ originalMerge(arenaArg, playerArg); beep(500,40); };
  const originalSweep = sweep;
  sweep = function(){ originalSweep(); beep(900,120); };

  // Make sure touch controls visible on small screens
  function showOrHideTouch(){ if(window.innerWidth <= 820) document.getElementById('touchControls').style.display = 'flex'; else document.getElementById('touchControls').style.display = 'none'; }
  showOrHideTouch(); window.addEventListener('resize', showOrHideTouch);
  </script></body>
</html>
