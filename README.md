<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover" />
  <title>Among Us Mini PeerJS</title>
  <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
  <style>
    * { box-sizing: border-box; }
    html, body {
      width: 100%;
      height: 100%;
      margin: 0;
      padding: 0;
      overflow: hidden;
      overscroll-behavior: none;
      touch-action: none;
      -webkit-user-select: none;
      user-select: none;
      -webkit-tap-highlight-color: transparent;
    }
    body {
      position: fixed;
      inset: 0;
      font-family: Arial, sans-serif;
      background: radial-gradient(circle at top, #18223a, #070914 65%);
      color: white;
    }
    button, input, select {
      font: inherit;
    }
    button {
      touch-action: manipulation;
      border: 0;
      border-radius: 12px;
      padding: 10px 14px;
      color: white;
      background: #3348ff;
      cursor: pointer;
      font-weight: 700;
      box-shadow: 0 6px 0 rgba(0,0,0,.25);
    }
    button:hover { filter: brightness(1.12); }
    button:active { transform: translateY(2px); box-shadow: 0 3px 0 rgba(0,0,0,.25); }
    button.danger { background: #e8273f; }
    button.warn { background: #f59e0b; }
    button.good { background: #16a34a; }
    button.ghost { background: rgba(255,255,255,.12); }
    button:disabled { opacity: .45; cursor: not-allowed; }
    input, select {
      user-select: text;
      -webkit-user-select: text;
      touch-action: manipulation;
      width: 100%;
      padding: 11px 12px;
      border-radius: 12px;
      border: 1px solid rgba(255,255,255,.18);
      background: rgba(255,255,255,.08);
      color: white;
      outline: none;
    }
    option { color: black; }
    .screen {
      position: fixed;
      inset: 0;
      height: 100dvh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: max(12px, env(safe-area-inset-top)) max(12px, env(safe-area-inset-right)) max(12px, env(safe-area-inset-bottom)) max(12px, env(safe-area-inset-left));
      background: rgba(4,7,18,.78);
      z-index: 10;
      overflow: hidden;
    }
    .hidden { display: none !important; }
    .panel {
      width: min(960px, 100%);
      max-height: calc(100dvh - 24px);
      overflow: auto;
      overscroll-behavior: contain;
      touch-action: pan-y;
      border: 1px solid rgba(255,255,255,.14);
      border-radius: 24px;
      background: linear-gradient(180deg, rgba(24,31,55,.96), rgba(10,13,27,.96));
      box-shadow: 0 25px 70px rgba(0,0,0,.55);
      padding: 22px;
    }
    .title {
      margin: 0 0 8px;
      font-size: clamp(28px, 5vw, 54px);
      line-height: 1;
      letter-spacing: -1px;
      text-shadow: 0 4px 0 rgba(0,0,0,.35);
    }
    .subtitle { margin: 0 0 18px; opacity: .8; }
    .grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 14px; }
    .card {
      border-radius: 18px;
      border: 1px solid rgba(255,255,255,.12);
      background: rgba(255,255,255,.07);
      padding: 15px;
    }
    .row { display: flex; gap: 10px; align-items: center; }
    .row > * { flex: 1; }
    .small { font-size: 13px; opacity: .76; }
    .pill {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      padding: 5px 9px;
      border-radius: 999px;
      background: rgba(255,255,255,.11);
      border: 1px solid rgba(255,255,255,.12);
      font-size: 12px;
      font-weight: 700;
    }
    #gameWrap {
      position: fixed;
      inset: 0;
      width: 100vw;
      height: 100dvh;
      display: grid;
      grid-template-columns: 1fr 330px;
      background: #080b17;
      overflow: hidden;
    }
    #canvasBox { position: relative; overflow: hidden; min-width: 0; min-height: 0; touch-action: none; }
    canvas { display: block; width: 100%; height: 100%; background: #0a1022; touch-action: none; }
    #hud {
      border-left: 1px solid rgba(255,255,255,.12);
      background: linear-gradient(180deg, #11182d, #090d19);
      padding: 14px;
      overflow: auto;
      overscroll-behavior: contain;
      touch-action: pan-y;
    }
    #roleBox {
      padding: 12px;
      border-radius: 16px;
      background: rgba(255,255,255,.08);
      border: 1px solid rgba(255,255,255,.13);
      margin-bottom: 12px;
    }
    #roleName { font-size: 24px; font-weight: 900; }
    #taskList { padding-left: 18px; line-height: 1.6; }
    #playersList, #logList, #voteList { display: grid; gap: 8px; }
    .playerItem, .logItem, .voteItem {
      padding: 9px;
      border-radius: 12px;
      background: rgba(255,255,255,.07);
      border: 1px solid rgba(255,255,255,.09);
    }
    .playerItem.dead { opacity: .45; text-decoration: line-through; }
    .sectionTitle { margin: 18px 0 8px; font-size: 14px; text-transform: uppercase; opacity: .75; letter-spacing: .06em; }
    #actionBar {
      position: absolute;
      left: calc(16px + env(safe-area-inset-left));
      bottom: calc(16px + env(safe-area-inset-bottom));
      right: 350px;
      display: flex;
      gap: 10px;
      pointer-events: none;
      z-index: 4;
    }
    #actionBar button { pointer-events: auto; min-width: 110px; }
    #mobilePad {
      position: absolute;
      right: calc(350px + 18px);
      bottom: calc(18px + env(safe-area-inset-bottom));
      display: none;
      width: 150px;
      height: 150px;
      border-radius: 50%;
      background: rgba(255,255,255,.08);
      border: 1px solid rgba(255,255,255,.14);
      pointer-events: auto;
      touch-action: none;
      z-index: 4;
    }
    #mobileStick {
      position: absolute;
      left: 50%;
      top: 50%;
      width: 58px;
      height: 58px;
      margin-left: -29px;
      margin-top: -29px;
      border-radius: 50%;
      background: rgba(255,255,255,.26);
      border: 2px solid rgba(255,255,255,.3);
      box-shadow: 0 8px 18px rgba(0,0,0,.25);
      pointer-events: none;
    }
    #mobilePadHint {
      position: absolute;
      inset: 0;
      display: grid;
      place-items: center;
      color: rgba(255,255,255,.5);
      font-size: 12px;
      font-weight: 900;
      pointer-events: none;
    }
    #toast {
      position: absolute;
      left: 50%;
      top: 16px;
      transform: translateX(-50%);
      background: rgba(0,0,0,.62);
      border: 1px solid rgba(255,255,255,.18);
      border-radius: 999px;
      padding: 10px 14px;
      z-index: 5;
      opacity: 0;
      transition: .2s;
      pointer-events: none;
      max-width: 70%;
      text-align: center;
    }
    #toast.show { opacity: 1; }
    .meetingHeader {
      display: flex;
      justify-content: space-between;
      gap: 12px;
      align-items: center;
      margin-bottom: 14px;
    }
    .voteItem { display: flex; gap: 8px; align-items: center; justify-content: space-between; }
    .voteItem span { flex: 1; }
    #copyId { user-select: all; color: #a5b4fc; font-weight: 800; }
    @media (max-width: 900px) {
      #gameWrap { grid-template-columns: 1fr; grid-template-rows: minmax(0, 1fr) 190px; }
      #hud {
        border-left: 0;
        border-top: 1px solid rgba(255,255,255,.12);
        padding: 10px 12px calc(10px + env(safe-area-inset-bottom));
      }
      #roleBox { margin-bottom: 8px; padding: 9px; }
      #roleName { font-size: 18px; }
      .sectionTitle { margin: 10px 0 6px; font-size: 11px; }
      #taskList { margin: 6px 0; line-height: 1.35; font-size: 12px; }
      #playersList, #logList { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 5px; }
      .playerItem, .logItem { padding: 6px; font-size: 12px; }
      #hud .card { padding: 8px; }
      #hud .row { gap: 6px; }
      #hud button { padding: 8px 10px; border-radius: 10px; }
      #actionBar {
        left: calc(10px + env(safe-area-inset-left));
        right: calc(10px + env(safe-area-inset-right));
        bottom: calc(202px + env(safe-area-inset-bottom));
        justify-content: flex-start;
      }
      #actionBar button { min-width: 86px; padding: 11px 10px; }
      #mobilePad {
        display: block;
        right: calc(12px + env(safe-area-inset-right));
        bottom: calc(258px + env(safe-area-inset-bottom));
      }
      .grid { grid-template-columns: 1fr; }
      .panel { padding: 14px; border-radius: 18px; }
      .title { font-size: clamp(25px, 8vw, 38px); }
      .meetingHeader { align-items: flex-start; }
      .voteItem { padding: 8px; }
    }
    @media (max-width: 520px) {
      #gameWrap { grid-template-rows: minmax(0, 1fr) 166px; }
      #hud { font-size: 12px; }
      #roleHint, #logList, #hostId { display: none; }
      #mobilePad { width: 132px; height: 132px; bottom: calc(224px + env(safe-area-inset-bottom)); }
      #mobileStick { width: 52px; height: 52px; margin-left: -26px; margin-top: -26px; }
      #actionBar { bottom: calc(174px + env(safe-area-inset-bottom)); gap: 7px; }
      #actionBar button { min-width: 76px; padding: 10px 8px; font-size: 13px; }
      #playersList { grid-template-columns: repeat(3, minmax(0, 1fr)); }
      #taskList li:nth-child(n+4) { display: none; }
    }
    @media (orientation: landscape) and (max-height: 520px) {
      #gameWrap { grid-template-columns: 1fr 260px; grid-template-rows: 1fr; }
      #hud { border-top: 0; border-left: 1px solid rgba(255,255,255,.12); padding: 8px; }
      #roleName { font-size: 17px; }
      #taskList { font-size: 12px; line-height: 1.28; }
      #logList { display: none; }
      #actionBar { right: 280px; bottom: calc(10px + env(safe-area-inset-bottom)); left: calc(10px + env(safe-area-inset-left)); }
      #mobilePad { right: calc(280px + 12px); bottom: calc(12px + env(safe-area-inset-bottom)); display: block; width: 126px; height: 126px; }
    }
  </style>
</head>
<body>
  <div id="gameWrap">
    <div id="canvasBox">
      <canvas id="game"></canvas>
      <div id="toast"></div>
      <div id="actionBar">
        <button id="btnUse" class="good">Dùng</button>
        <button id="btnKill" class="danger">Kill</button>
        <button id="btnReport" class="warn">Report</button>
      </div>
      <div id="mobilePad" aria-label="Cần điều khiển di chuyển">
        <div id="mobilePadHint">MOVE</div>
        <div id="mobileStick"></div>
      </div>
    </div>
    <aside id="hud">
      <div id="roleBox">
        <div class="small">Vai trò của bạn</div>
        <div id="roleName">Chưa bắt đầu</div>
        <div id="roleHint" class="small">Tạo hoặc vào phòng để chơi.</div>
      </div>
      <div class="row">
        <button id="btnStart" class="good">Start Game</button>
        <button id="btnEmergency" class="danger">Emergency</button>
      </div>
      <div class="sectionTitle">Phòng</div>
      <div class="card small">
        Peer ID: <span id="myId">...</span><br />
        Host: <span id="hostId">...</span><br />
        Trạng thái: <span id="netStatus">offline</span>
      </div>
      <div class="sectionTitle">Task</div>
      <ul id="taskList"></ul>
      <div class="sectionTitle">Người chơi</div>
      <div id="playersList"></div>
      <div class="sectionTitle">Nhật ký</div>
      <div id="logList"></div>
    </aside>
  </div>

  <div id="menu" class="screen">
    <div class="panel">
      <h1 class="title">Among Us Mini<br />PeerJS</h1>
      <p class="subtitle">Game nhiều người chơi P2P: host tạo phòng, người khác nhập Peer ID để vào. Không cần server riêng.</p>
      <div class="grid">
        <div class="card">
          <h2>Tạo nhân vật</h2>
          <label class="small">Tên</label>
          <input id="nameInput" maxlength="14" placeholder="VD: Hai" />
          <br /><br />
          <label class="small">Màu</label>
          <select id="colorInput">
            <option value="#ef4444">Đỏ</option>
            <option value="#3b82f6">Xanh dương</option>
            <option value="#22c55e">Xanh lá</option>
            <option value="#facc15">Vàng</option>
            <option value="#a855f7">Tím</option>
            <option value="#f97316">Cam</option>
            <option value="#ec4899">Hồng</option>
            <option value="#14b8a6">Ngọc</option>
          </select>
          <br /><br />
          <button id="btnCreatePeer">Kết nối PeerJS</button>
          <p class="small">Bước 1: bấm kết nối để lấy Peer ID của bạn.</p>
        </div>
        <div class="card">
          <h2>Host / Join</h2>
          <button id="btnHost" class="good" disabled>Tạo phòng</button>
          <p class="small">Gửi Peer ID của bạn cho bạn bè. ID phòng: <span id="copyId">chưa có</span></p>
          <div class="row">
            <input id="joinIdInput" placeholder="Nhập Peer ID của host" />
            <button id="btnJoin" disabled>Vào</button>
          </div>
          <p class="small">Host là máy điều khiển trạng thái game: chia role, vote, thắng/thua.</p>
        </div>
      </div>
      <div class="card" style="margin-top:14px">
        <b>Luật nhanh:</b>
        <span class="pill">WASD / phím mũi tên để đi</span>
        <span class="pill">Crewmate làm task</span>
        <span class="pill">Impostor kill</span>
        <span class="pill">Report để vote</span>
        <span class="pill">Crew thắng khi xong task hoặc đuổi Impostor</span>
      </div>
    </div>
  </div>

  <div id="meeting" class="screen hidden">
    <div class="panel">
      <div class="meetingHeader">
        <div>
          <h1 class="title" style="font-size:38px">Cuộc họp khẩn</h1>
          <p id="meetingReason" class="subtitle">Ai là Impostor?</p>
        </div>
        <div class="pill">Còn <span id="meetingTimer">30</span>s</div>
      </div>
      <div id="voteList"></div>
      <br />
      <button id="btnSkip" class="ghost">Skip vote</button>
    </div>
  </div>

  <div id="endScreen" class="screen hidden">
    <div class="panel" style="text-align:center">
      <h1 id="endTitle" class="title">Game Over</h1>
      <p id="endText" class="subtitle"></p>
      <button onclick="location.reload()">Chơi lại</button>
    </div>
  </div>

<script>
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const $ = id => document.getElementById(id);

  const WORLD = { w: 1600, h: 1000 };
  const SPEED = 230;
  const KILL_RANGE = 75;
  const REPORT_RANGE = 90;
  const TASK_RANGE = 70;
  const KILL_COOLDOWN = 12000;
  const MEETING_SECONDS = 30;

  let peer = null;
  let myId = null;
  let host = false;
  let hostPeerId = null;
  let conns = new Map();
  let hostConn = null;
  let lastPing = 0;
  let keys = {};
  let mobile = { x: 0, y: 0, active: false, pointerId: null };
  let lastFrame = performance.now();
  let toastTimer = null;
  let meetingInterval = null;

  const state = {
    phase: 'menu',
    players: {},
    corpses: [],
    tasks: makeTasks(),
    votes: {},
    meeting: null,
    winner: null,
    log: []
  };

  const walls = [
    {x:0,y:0,w:1600,h:24},{x:0,y:976,w:1600,h:24},{x:0,y:0,w:24,h:1000},{x:1576,y:0,w:24,h:1000},
    {x:250,y:120,w:280,h:160, name:'Cafeteria'},
    {x:720,y:80,w:360,h:180, name:'MedBay'},
    {x:1160,y:150,w:280,h:190, name:'Admin'},
    {x:165,y:470,w:310,h:230, name:'Engine'},
    {x:650,y:440,w:330,h:240, name:'Electrical'},
    {x:1120,y:520,w:310,h:230, name:'Security'},
    {x:470,y:312,w:80,h:320},
    {x:1010,y:300,w:80,h:360},
  ];

  function makeTasks() {
    return [
      { id:'wires', name:'Nối dây điện', x:815, y:725, kind:'hold', progress:0, total:2.2 },
      { id:'scan', name:'Quét y tế', x:900, y:310, kind:'hold', progress:0, total:2.8 },
      { id:'card', name:'Quẹt thẻ Admin', x:1270, y:390, kind:'tap', progress:0, total:1 },
      { id:'fuel', name:'Đổ nhiên liệu', x:322, y:760, kind:'hold', progress:0, total:2.5 },
      { id:'download', name:'Tải dữ liệu', x:1380, y:800, kind:'tap', progress:0, total:1 }
    ];
  }

  function me() { return state.players[myId]; }
  function randSpawn() { return { x: 780 + Math.random()*80, y: 880 + Math.random()*50 }; }
  function safeName() { return ($('nameInput').value || 'Player').trim().slice(0,14) || 'Player'; }
  function myColor() { return $('colorInput').value; }

  function resize() {
    const box = $('canvasBox').getBoundingClientRect();
    const ratio = Math.min(window.devicePixelRatio || 1, 2);
    canvas.width = Math.max(300, Math.floor(box.width * ratio));
    canvas.height = Math.max(250, Math.floor(box.height * ratio));
    ctx.setTransform(ratio,0,0,ratio,0,0);
  }
  addEventListener('resize', resize, { passive: true });
  addEventListener('orientationchange', () => setTimeout(resize, 250), { passive: true });
  resize();

  function toast(msg) {
    $('toast').textContent = msg;
    $('toast').classList.add('show');
    clearTimeout(toastTimer);
    toastTimer = setTimeout(() => $('toast').classList.remove('show'), 2300);
  }

  function addLog(msg) {
    state.log.unshift({ t: Date.now(), msg });
    state.log = state.log.slice(0, 8);
  }

  function createLocalPlayer() {
    const p = randSpawn();
    state.players[myId] = {
      id: myId,
      name: safeName(),
      color: myColor(),
      x: p.x,
      y: p.y,
      role: 'unknown',
      alive: true,
      connected: true,
      tasksDone: {},
      killReadyAt: 0,
      emergencyUsed: false
    };
  }

  function connectPeer() {
    if (peer) return;
    peer = new Peer();
    $('netStatus').textContent = 'đang kết nối...';

    peer.on('open', id => {
      myId = id;
      $('myId').textContent = id;
      $('copyId').textContent = id;
      $('netStatus').textContent = 'online';
      $('btnHost').disabled = false;
      $('btnJoin').disabled = false;
      createLocalPlayer();
      updateHUD();
      toast('Đã kết nối PeerJS. Bạn có thể tạo phòng hoặc vào phòng.');
    });

    peer.on('connection', conn => {
      if (!host) {
        conn.close();
        return;
      }
      setupHostConnection(conn);
    });

    peer.on('error', err => {
      console.error(err);
      toast('Lỗi PeerJS: ' + err.type);
      $('netStatus').textContent = 'lỗi';
    });
  }

  function setupHostConnection(conn) {
    conns.set(conn.peer, conn);
    conn.on('open', () => {
      conn.send({ type:'welcome', hostId: myId, state, yourId: conn.peer });
      broadcast({ type:'state', state });
    });
    conn.on('data', msg => handleHostMessage(conn.peer, msg));
    conn.on('close', () => {
      conns.delete(conn.peer);
      if (state.players[conn.peer]) state.players[conn.peer].connected = false;
      addLog((state.players[conn.peer]?.name || conn.peer) + ' đã rời phòng');
      broadcastState();
    });
  }

  function joinHost() {
    const id = $('joinIdInput').value.trim();
    if (!peer || !id || id === myId) return toast('Peer ID không hợp lệ.');
    host = false;
    hostPeerId = id;
    $('hostId').textContent = id;
    hostConn = peer.connect(id, { reliable: true });
    hostConn.on('open', () => {
      hostConn.send({ type:'join', player: state.players[myId] });
      $('menu').classList.add('hidden');
      toast('Đã vào phòng. Chờ host start game.');
    });
    hostConn.on('data', handleClientMessage);
    hostConn.on('close', () => toast('Mất kết nối với host.'));
    hostConn.on('error', () => toast('Không vào được phòng.'));
  }

  function becomeHost() {
    host = true;
    hostPeerId = myId;
    $('hostId').textContent = myId;
    $('menu').classList.add('hidden');
    addLog('Bạn đã tạo phòng');
    toast('Bạn là host. Gửi Peer ID cho bạn bè rồi bấm Start Game.');
    updateHUD();
  }

  function sendToHost(msg) {
    if (host) handleHostMessage(myId, msg);
    else if (hostConn && hostConn.open) hostConn.send(msg);
  }

  function broadcast(msg) {
    for (const conn of conns.values()) if (conn.open) conn.send(msg);
  }

  function broadcastState() {
    broadcast({ type:'state', state });
    updateHUD();
  }

  function handleHostMessage(from, msg) {
    if (!msg || !msg.type) return;
    if (msg.type === 'join') {
      const p = msg.player;
      if (!p) return;
      const spawn = randSpawn();
      state.players[from] = { ...p, id: from, x: spawn.x, y: spawn.y, role:'unknown', alive:true, connected:true, tasksDone:{}, killReadyAt:0, emergencyUsed:false };
      addLog(state.players[from].name + ' đã vào phòng');
      broadcastState();
    }
    if (msg.type === 'input') {
      const p = state.players[from];
      if (!p || state.phase !== 'playing' || !p.alive) return;
      movePlayer(p, msg.dx || 0, msg.dy || 0, msg.dt || 0);
      p.x = clamp(p.x, 34, WORLD.w-34);
      p.y = clamp(p.y, 34, WORLD.h-34);
    }
    if (msg.type === 'task') doTask(from, msg.taskId, msg.dt || 0, msg.mode);
    if (msg.type === 'kill') doKill(from, msg.targetId);
    if (msg.type === 'report') doReport(from, msg.corpseId || null, msg.emergency || false);
    if (msg.type === 'vote') doVote(from, msg.targetId);
  }

  function handleClientMessage(msg) {
    if (!msg || !msg.type) return;
    if (msg.type === 'welcome') {
      hostPeerId = msg.hostId;
      Object.assign(state, msg.state);
      $('hostId').textContent = hostPeerId;
      updateHUD();
    }
    if (msg.type === 'state') {
      Object.assign(state, msg.state);
      updateMeetingUI();
      updateHUD();
      if (state.winner) showEnd();
    }
    if (msg.type === 'toast') toast(msg.msg);
  }

  function startGame() {
    if (!host) return toast('Chỉ host mới start được.');
    const ids = Object.keys(state.players).filter(id => state.players[id].connected !== false);
    if (ids.length < 3) return toast('Cần ít nhất 3 người để chơi hay hơn. Có thể sửa code nếu muốn 2 người.');
    state.phase = 'playing';
    state.corpses = [];
    state.tasks = makeTasks();
    state.votes = {};
    state.meeting = null;
    state.winner = null;
    const impostorCount = ids.length >= 7 ? 2 : 1;
    const shuffled = [...ids].sort(() => Math.random() - .5);
    const impostors = new Set(shuffled.slice(0, impostorCount));
    ids.forEach((id, i) => {
      const spawn = { x: 690 + (i%5)*58, y: 860 + Math.floor(i/5)*55 };
      state.players[id] = {
        ...state.players[id],
        x: spawn.x,
        y: spawn.y,
        role: impostors.has(id) ? 'impostor' : 'crewmate',
        alive: true,
        connected: true,
        tasksDone: {},
        killReadyAt: Date.now() + 5000,
        emergencyUsed: false
      };
    });
    addLog('Game bắt đầu. Có ' + impostorCount + ' impostor.');
    broadcastState();
    updateHUD();
  }

  function doTask(playerId, taskId, dt, mode) {
    const p = state.players[playerId];
    const t = state.tasks.find(x => x.id === taskId);
    if (!p || !t || p.role !== 'crewmate' || !p.alive || state.phase !== 'playing') return;
    if (dist(p, t) > TASK_RANGE) return;
    if (p.tasksDone[taskId]) return;
    if (t.kind === 'tap' || mode === 'tap') {
      p.tasksDone[taskId] = true;
      addLog(p.name + ' hoàn thành task: ' + t.name);
    } else {
      const key = playerId + ':' + taskId;
      t.progress = t.progress || {};
      t.progress[key] = (t.progress[key] || 0) + dt;
      if (t.progress[key] >= t.total) {
        p.tasksDone[taskId] = true;
        addLog(p.name + ' hoàn thành task: ' + t.name);
      }
    }
    checkWin();
    broadcastState();
  }

  function doKill(killerId, targetId) {
    const k = state.players[killerId];
    const v = state.players[targetId];
    if (!k || !v || k.role !== 'impostor' || !k.alive || !v.alive || v.role === 'impostor') return;
    if (state.phase !== 'playing') return;
    if (Date.now() < k.killReadyAt) return;
    if (dist(k, v) > KILL_RANGE) return;
    v.alive = false;
    state.corpses.push({ id:'corpse_' + Date.now(), playerId: targetId, x:v.x, y:v.y, color:v.color, name:v.name });
    k.killReadyAt = Date.now() + KILL_COOLDOWN;
    addLog('Một xác đã xuất hiện...');
    checkWin();
    broadcastState();
  }

  function doReport(reporterId, corpseId, emergency) {
    const p = state.players[reporterId];
    if (!p || !p.alive || state.phase !== 'playing') return;
    if (emergency) {
      if (p.emergencyUsed) return;
      p.emergencyUsed = true;
    } else {
      const c = state.corpses.find(x => x.id === corpseId);
      if (!c || dist(p, c) > REPORT_RANGE) return;
    }
    state.phase = 'meeting';
    state.meeting = { reason: emergency ? p.name + ' bấm Emergency Meeting' : p.name + ' report xác', endAt: Date.now() + MEETING_SECONDS * 1000 };
    state.votes = {};
    state.corpses = [];
    addLog(state.meeting.reason);
    broadcastState();
    startMeetingTimerHost();
  }

  function doVote(voterId, targetId) {
    const p = state.players[voterId];
    if (!p || !p.alive || state.phase !== 'meeting') return;
    state.votes[voterId] = targetId || 'skip';
    const alive = Object.values(state.players).filter(x => x.alive).length;
    if (Object.keys(state.votes).length >= alive) finishMeeting();
    else broadcastState();
  }

  function startMeetingTimerHost() {
    clearInterval(meetingInterval);
    meetingInterval = setInterval(() => {
      if (!host || state.phase !== 'meeting' || !state.meeting) return clearInterval(meetingInterval);
      if (Date.now() >= state.meeting.endAt) finishMeeting();
      else broadcastState();
    }, 1000);
  }

  function finishMeeting() {
    if (!host || state.phase !== 'meeting') return;
    clearInterval(meetingInterval);
    const counts = {};
    Object.values(state.votes).forEach(v => counts[v] = (counts[v] || 0) + 1);
    let top = 'skip', topCount = 0, tie = false;
    for (const [target, count] of Object.entries(counts)) {
      if (count > topCount) { top = target; topCount = count; tie = false; }
      else if (count === topCount) tie = true;
    }
    if (!topCount || tie || top === 'skip') {
      addLog('Không ai bị đuổi.');
    } else if (state.players[top]) {
      state.players[top].alive = false;
      addLog(state.players[top].name + ' bị đuổi. Vai trò: ' + roleVN(state.players[top].role));
    }
    state.phase = 'playing';
    state.meeting = null;
    state.votes = {};
    Object.values(state.players).forEach(p => p.killReadyAt = Math.max(p.killReadyAt || 0, Date.now() + 3500));
    checkWin();
    broadcastState();
  }

  function checkWin() {
    const alive = Object.values(state.players).filter(p => p.alive);
    const impAlive = alive.filter(p => p.role === 'impostor').length;
    const crewAlive = alive.filter(p => p.role === 'crewmate').length;
    if (impAlive === 0) return endGame('crew', 'Crewmate thắng! Tất cả Impostor đã bị loại.');
    if (impAlive >= crewAlive) return endGame('impostor', 'Impostor thắng! Số Impostor đã áp đảo crew.');

    const crews = Object.values(state.players).filter(p => p.role === 'crewmate');
    const total = crews.length * state.tasks.length;
    const done = crews.reduce((sum, p) => sum + Object.keys(p.tasksDone || {}).length, 0);
    if (total > 0 && done >= total) return endGame('crew', 'Crewmate thắng! Tất cả task đã hoàn thành.');
  }

  function endGame(team, text) {
    state.phase = 'ended';
    state.winner = { team, text };
    addLog(text);
  }

  function showEnd() {
    $('endTitle').textContent = state.winner.team === 'crew' ? 'CREWMATE WIN' : 'IMPOSTOR WIN';
    $('endText').textContent = state.winner.text;
    $('endScreen').classList.remove('hidden');
  }

  function roleVN(r) { return r === 'impostor' ? 'Impostor' : r === 'crewmate' ? 'Crewmate' : 'Chưa rõ'; }

  function movePlayer(p, dx, dy, dt) {
    if (!dx && !dy) return;
    const len = Math.hypot(dx, dy) || 1;
    let nx = p.x + dx / len * SPEED * dt;
    let ny = p.y + dy / len * SPEED * dt;
    const old = { x:p.x, y:p.y };
    p.x = nx;
    if (collidesPlayer(p)) p.x = old.x;
    p.y = ny;
    if (collidesPlayer(p)) p.y = old.y;
  }

  function collidesPlayer(p) {
    const r = 20;
    return walls.some(w => circleRect(p.x, p.y, r, w));
  }

  function circleRect(cx, cy, cr, r) {
    const x = clamp(cx, r.x, r.x + r.w);
    const y = clamp(cy, r.y, r.y + r.h);
    return Math.hypot(cx - x, cy - y) < cr;
  }

  function clamp(v, a, b) { return Math.max(a, Math.min(b, v)); }
  function dist(a, b) { return Math.hypot(a.x - b.x, a.y - b.y); }

  function nearestTask() {
    const p = me();
    if (!p || !p.alive || p.role !== 'crewmate') return null;
    return state.tasks.find(t => !p.tasksDone[t.id] && dist(p, t) <= TASK_RANGE) || null;
  }

  function nearestVictim() {
    const p = me();
    if (!p || !p.alive || p.role !== 'impostor') return null;
    return Object.values(state.players).find(v => v.id !== myId && v.alive && v.role === 'crewmate' && dist(p, v) <= KILL_RANGE) || null;
  }

  function nearestCorpse() {
    const p = me();
    if (!p || !p.alive) return null;
    return state.corpses.find(c => dist(p, c) <= REPORT_RANGE) || null;
  }

  function localInput(dt) {
    if (!myId || state.phase !== 'playing') return;
    const p = me();
    if (!p || !p.alive) return;
    let dx = 0, dy = 0;
    if (keys.KeyA || keys.ArrowLeft) dx--;
    if (keys.KeyD || keys.ArrowRight) dx++;
    if (keys.KeyW || keys.ArrowUp) dy--;
    if (keys.KeyS || keys.ArrowDown) dy++;
    dx += mobile.x || 0;
    dy += mobile.y || 0;
    if (dx || dy) {
      if (host) {
        movePlayer(p, dx, dy, dt);
        broadcastStateThrottled();
      } else {
        sendToHost({ type:'input', dx, dy, dt });
      }
    }
    if (keys.Space) {
      const t = nearestTask();
      if (t && t.kind === 'hold') sendToHost({ type:'task', taskId:t.id, dt, mode:'hold' });
    }
  }

  let lastBroadcast = 0;
  function broadcastStateThrottled() {
    if (!host) return;
    const now = performance.now();
    if (now - lastBroadcast > 45) {
      lastBroadcast = now;
      broadcastState();
    }
  }

  function useAction() {
    const t = nearestTask();
    if (!t) return toast('Không có task gần bạn.');
    sendToHost({ type:'task', taskId:t.id, dt:0.2, mode: t.kind === 'tap' ? 'tap' : 'hold' });
  }
  function killAction() {
    const p = me();
    const v = nearestVictim();
    if (!p || p.role !== 'impostor') return toast('Bạn không phải Impostor.');
    if (Date.now() < p.killReadyAt) return toast('Kill cooldown còn ' + Math.ceil((p.killReadyAt - Date.now())/1000) + 's');
    if (!v) return toast('Không có crewmate đủ gần.');
    sendToHost({ type:'kill', targetId:v.id });
  }
  function reportAction() {
    const c = nearestCorpse();
    if (!c) return toast('Không có xác gần bạn.');
    sendToHost({ type:'report', corpseId:c.id });
  }
  function emergencyAction() {
    const p = me();
    if (!p || !p.alive || p.emergencyUsed) return toast('Bạn đã dùng Emergency hoặc đang chết.');
    const button = { x: 800, y: 885 };
    if (dist(p, button) > 90) return toast('Hãy đứng gần nút Emergency ở sảnh dưới.');
    sendToHost({ type:'report', emergency:true });
  }

  function draw() {
    const box = $('canvasBox').getBoundingClientRect();
    const w = box.width, h = box.height;
    ctx.clearRect(0, 0, w, h);
    const p = me() || { x:800, y:500 };
    const camX = WORLD.w <= w ? 0 : clamp(p.x - w/2, 0, WORLD.w - w);
    const camY = WORLD.h <= h ? 0 : clamp(p.y - h/2, 0, WORLD.h - h);
    ctx.save();
    ctx.translate(-camX, -camY);
    drawWorld();
    ctx.restore();
  }

  function drawWorld() {
    ctx.fillStyle = '#0d1730';
    ctx.fillRect(0, 0, WORLD.w, WORLD.h);
    drawGrid();
    walls.forEach(drawRoom);
    drawTaskObjects();
    drawEmergencyButton();
    state.corpses.forEach(drawCorpse);
    Object.values(state.players).forEach(drawPlayer);
  }

  function drawGrid() {
    ctx.strokeStyle = 'rgba(255,255,255,.035)';
    ctx.lineWidth = 1;
    for (let x=0; x<WORLD.w; x+=80) { ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,WORLD.h); ctx.stroke(); }
    for (let y=0; y<WORLD.h; y+=80) { ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(WORLD.w,y); ctx.stroke(); }
  }

  function drawRoom(r) {
    ctx.fillStyle = r.name ? '#18294d' : '#26365f';
    ctx.strokeStyle = 'rgba(255,255,255,.15)';
    ctx.lineWidth = 2;
    roundRect(r.x,r.y,r.w,r.h,18,true,true);
    if (r.name) {
      ctx.fillStyle = 'rgba(255,255,255,.68)';
      ctx.font = 'bold 16px Arial';
      ctx.fillText(r.name, r.x + 16, r.y + 28);
    }
  }

  function drawTaskObjects() {
    state.tasks.forEach(t => {
      const done = me()?.tasksDone?.[t.id];
      ctx.fillStyle = done ? '#22c55e' : '#facc15';
      ctx.beginPath(); ctx.arc(t.x, t.y, 16, 0, Math.PI*2); ctx.fill();
      ctx.strokeStyle = 'rgba(0,0,0,.45)'; ctx.lineWidth = 4; ctx.stroke();
      ctx.fillStyle = 'white'; ctx.font = '12px Arial'; ctx.textAlign = 'center';
      ctx.fillText(t.name, t.x, t.y - 24);
      ctx.textAlign = 'left';
    });
  }

  function drawEmergencyButton() {
    ctx.fillStyle = '#e11d48';
    roundRect(750, 850, 100, 70, 16, true, true);
    ctx.fillStyle = 'white';
    ctx.font = 'bold 13px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('EMERGENCY', 800, 880);
    ctx.fillText('BUTTON', 800, 898);
    ctx.textAlign = 'left';
  }

  function drawCorpse(c) {
    ctx.save();
    ctx.translate(c.x, c.y);
    ctx.fillStyle = c.color;
    ctx.beginPath(); ctx.ellipse(-8, 4, 22, 13, -0.3, 0, Math.PI*2); ctx.fill();
    ctx.fillStyle = '#b91c1c';
    ctx.beginPath(); ctx.arc(14, -2, 11, 0, Math.PI*2); ctx.fill();
    ctx.strokeStyle = 'white'; ctx.lineWidth = 3;
    ctx.beginPath(); ctx.moveTo(9, -7); ctx.lineTo(19, 3); ctx.moveTo(19, -7); ctx.lineTo(9, 3); ctx.stroke();
    ctx.restore();
  }

  function drawPlayer(p) {
    const ghost = !p.alive;
    ctx.save();
    ctx.globalAlpha = ghost ? .35 : 1;
    ctx.translate(p.x, p.y);
    ctx.fillStyle = p.color;
    roundRect(-18, -26, 36, 52, 17, true, false);
    ctx.fillStyle = shade(p.color, -30);
    roundRect(7, 0, 15, 24, 8, true, false);
    ctx.fillStyle = '#9be8ff';
    roundRect(-6, -18, 26, 14, 8, true, false);
    ctx.strokeStyle = 'rgba(0,0,0,.4)';
    ctx.lineWidth = 3;
    roundRect(-18, -26, 36, 52, 17, false, true);
    ctx.fillStyle = 'white';
    ctx.font = 'bold 13px Arial';
    ctx.textAlign = 'center';
    ctx.fillText(p.name, 0, -35);
    if (p.id === myId) {
      ctx.strokeStyle = '#facc15'; ctx.lineWidth = 3;
      ctx.beginPath(); ctx.arc(0, 0, 32, 0, Math.PI*2); ctx.stroke();
    }
    ctx.restore();
    ctx.textAlign = 'left';
  }

  function roundRect(x,y,w,h,r,fill,stroke) {
    ctx.beginPath();
    ctx.moveTo(x+r,y);
    ctx.arcTo(x+w,y,x+w,y+h,r);
    ctx.arcTo(x+w,y+h,x,y+h,r);
    ctx.arcTo(x,y+h,x,y,r);
    ctx.arcTo(x,y,x+w,y,r);
    if (fill) ctx.fill();
    if (stroke) ctx.stroke();
  }

  function shade(hex, amt) {
    const n = parseInt(hex.slice(1), 16);
    let r = (n >> 16) + amt, g = ((n >> 8) & 255) + amt, b = (n & 255) + amt;
    r = clamp(r,0,255); g = clamp(g,0,255); b = clamp(b,0,255);
    return '#' + ((1<<24) + (r<<16) + (g<<8) + b).toString(16).slice(1);
  }

  function updateHUD() {
    const p = me();
    if (p) {
      $('roleName').textContent = roleVN(p.role);
      $('roleName').style.color = p.role === 'impostor' ? '#ef4444' : p.role === 'crewmate' ? '#22c55e' : 'white';
      $('roleHint').textContent = p.role === 'impostor'
        ? 'Giả vờ làm task, hạ crew và tránh bị vote.'
        : p.role === 'crewmate'
          ? 'Hoàn thành task, report xác, tìm Impostor.'
          : 'Chờ host bắt đầu game.';
    }
    $('btnStart').disabled = !host || state.phase === 'playing' || state.phase === 'meeting';
    $('btnKill').style.display = p?.role === 'impostor' ? 'inline-block' : 'none';
    $('btnUse').style.display = p?.role === 'crewmate' ? 'inline-block' : 'none';
    $('btnReport').disabled = !nearestCorpse();
    $('btnEmergency').disabled = !p || !p.alive || p.emergencyUsed || state.phase !== 'playing';

    const crews = Object.values(state.players).filter(x => x.role === 'crewmate');
    const total = Math.max(1, crews.length * state.tasks.length);
    const done = crews.reduce((sum, x) => sum + Object.keys(x.tasksDone || {}).length, 0);
    $('taskList').innerHTML = state.tasks.map(t => {
      const d = p?.tasksDone?.[t.id];
      return `<li>${d ? '✅' : '⬜'} ${t.name}</li>`;
    }).join('') + `<li><b>Tiến độ team:</b> ${done}/${total}</li>`;

    $('playersList').innerHTML = Object.values(state.players).map(x => `
      <div class="playerItem ${x.alive ? '' : 'dead'}">
        <b style="color:${x.color}">${escapeHtml(x.name)}</b>
        <span class="small">${x.alive ? 'sống' : 'chết'} ${x.connected === false ? ' · offline' : ''}</span>
      </div>`).join('');

    $('logList').innerHTML = state.log.map(l => `<div class="logItem small">${escapeHtml(l.msg)}</div>`).join('');
  }

  function updateMeetingUI() {
    if (state.phase === 'meeting') {
      $('meeting').classList.remove('hidden');
      $('meetingReason').textContent = state.meeting?.reason || 'Cuộc họp';
      const remain = Math.max(0, Math.ceil(((state.meeting?.endAt || Date.now()) - Date.now())/1000));
      $('meetingTimer').textContent = remain;
      const alive = Object.values(state.players).filter(p => p.alive);
      $('voteList').innerHTML = alive.map(p => `
        <div class="voteItem">
          <span><b style="color:${p.color}">${escapeHtml(p.name)}</b> <span class="small">${votesFor(p.id)} vote</span></span>
          <button data-vote="${p.id}" ${!me()?.alive || state.votes[myId] ? 'disabled' : ''}>Vote</button>
        </div>
      `).join('');
      document.querySelectorAll('[data-vote]').forEach(b => b.onclick = () => sendToHost({ type:'vote', targetId:b.dataset.vote }));
    } else {
      $('meeting').classList.add('hidden');
    }
  }

  function votesFor(id) { return Object.values(state.votes).filter(v => v === id).length; }
  function escapeHtml(str) { return String(str).replace(/[&<>"]/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c])); }

  function tick(now) {
    const dt = Math.min(.05, (now - lastFrame) / 1000);
    lastFrame = now;
    localInput(dt);
    draw();
    if (host && now - lastPing > 800) {
      lastPing = now;
      if (state.phase === 'playing') checkWin();
      broadcastState();
    }
    requestAnimationFrame(tick);
  }
  requestAnimationFrame(tick);

  addEventListener('keydown', e => {
    keys[e.code] = true;
    if (['Space','ArrowUp','ArrowDown','ArrowLeft','ArrowRight'].includes(e.code)) e.preventDefault();
  });
  addEventListener('keyup', e => keys[e.code] = false);

  function lockMobilePage() {
    const stop = e => {
      if (e.target.closest('.panel, #hud, input, select')) return;
      e.preventDefault();
    };
    document.addEventListener('touchmove', stop, { passive: false });
    document.addEventListener('gesturestart', e => e.preventDefault(), { passive: false });
  }
  lockMobilePage();

  const pad = $('mobilePad');
  const stick = $('mobileStick');
  function setStick(clientX, clientY) {
    const r = pad.getBoundingClientRect();
    const cx = r.left + r.width / 2;
    const cy = r.top + r.height / 2;
    const max = r.width * 0.32;
    let x = clientX - cx;
    let y = clientY - cy;
    const len = Math.hypot(x, y);
    if (len > max) { x = x / len * max; y = y / len * max; }
    mobile.x = Math.abs(x) < 6 ? 0 : x / max;
    mobile.y = Math.abs(y) < 6 ? 0 : y / max;
    stick.style.transform = `translate(${x}px, ${y}px)`;
  }
  function resetStick() {
    mobile.x = 0;
    mobile.y = 0;
    mobile.active = false;
    mobile.pointerId = null;
    stick.style.transform = 'translate(0, 0)';
  }
  pad.addEventListener('pointerdown', e => {
    mobile.active = true;
    mobile.pointerId = e.pointerId;
    pad.setPointerCapture(e.pointerId);
    setStick(e.clientX, e.clientY);
  });
  pad.addEventListener('pointermove', e => {
    if (!mobile.active || e.pointerId !== mobile.pointerId) return;
    setStick(e.clientX, e.clientY);
  });
  pad.addEventListener('pointerup', resetStick);
  pad.addEventListener('pointercancel', resetStick);

  $('btnCreatePeer').onclick = connectPeer;
  $('btnHost').onclick = becomeHost;
  $('btnJoin').onclick = joinHost;
  $('btnStart').onclick = startGame;
  $('btnUse').onclick = useAction;
  $('btnKill').onclick = killAction;
  $('btnReport').onclick = reportAction;
  $('btnEmergency').onclick = emergencyAction;
  $('btnSkip').onclick = () => sendToHost({ type:'vote', targetId:'skip' });
  $('copyId').onclick = async () => {
    try { await navigator.clipboard.writeText(myId || ''); toast('Đã copy Peer ID.'); } catch {}
  };

  document.addEventListener('visibilitychange', () => {
    if (document.hidden) resetStick();
  });

  setInterval(() => {
    updateHUD();
    updateMeetingUI();
  }, 350);

})();
</script>
</body>
</html>
    .stat-box span { display: block; color: var(--muted); font-size: 14px; }
    .stat-box strong { display: block; color: #d8b4fe; font-size: 30px; margin: 8px 0; word-break: break-word; }
    small { display: block; color: var(--muted); }
    .section-header { display: flex; justify-content: space-between; align-items: flex-start; gap: 16px; margin-bottom: 18px; }
    .section-header p { margin: 4px 0 0; color: var(--muted); }
    .form-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
    .connect-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    .tool-row { margin: 16px 0; }
    .tool-row input[type="range"] { width: 220px; padding: 0; }
    .role-grid, .player-grid, .action-grid { display: grid; gap: 10px; }
    .role-grid { grid-template-columns: repeat(4, 1fr); }
    .player-grid { grid-template-columns: repeat(3, 1fr); }
    .action-grid { grid-template-columns: repeat(3, 1fr); margin: 16px 0; }
    .role-mini span { font-size: 25px; }
    .role-mini b { display: block; margin-top: 6px; }
    .checkbox-line { display: flex; align-items: center; gap: 10px; background: var(--panel); border: 1px solid var(--line); border-radius: 20px; padding: 14px; margin: 14px 0; }
    .checkbox-line input { width: auto; }
    .winner-box { background: rgba(16,185,129,.14); border: 1px solid rgba(110,231,183,.3); border-radius: 24px; padding: 22px; }
    .winner-box h2 { color: #a7f3d0; font-size: 28px; }
    .seer-result { color: #d8b4fe; }
    .vote-list { display: flex; flex-direction: column; gap: 10px; }
    .vote-row { display: grid; grid-template-columns: 1fr 2fr; gap: 10px; align-items: center; background: var(--panel); border: 1px solid var(--line); border-radius: 18px; padding: 12px; }
    .player-card.dead { background: rgba(127,29,29,.26); border-color: rgba(248,113,113,.35); opacity: .74; }
    .player-top { display: flex; justify-content: space-between; gap: 12px; }
    .player-top b { font-size: 18px; }
    .player-top span { font-size: 28px; }
    .role-box { margin-top: 12px; background: #0f172a; border-radius: 16px; padding: 12px; }
    .role-box p { color: var(--muted); font-size: 13px; margin-bottom: 0; }
    .chat-box { height: 230px; overflow: auto; background: var(--panel); border: 1px solid var(--line); border-radius: 18px; padding: 12px; margin-bottom: 10px; }
    .chat-box p { margin: 0 0 8px; color: var(--soft); }
    .chat-box b { color: #d8b4fe; }
    .log-list { display: flex; flex-direction: column; gap: 10px; }
    .log-list div { color: var(--soft); font-size: 14px; }
    .compact-note { color: var(--muted); font-size: 13px; margin-top: 10px; }
    .hidden { display: none !important; }
    @media (max-width: 1050px) { .layout { grid-template-columns: 1fr; } .grid-3, .role-grid, .player-grid { grid-template-columns: repeat(2, 1fr); } .form-grid, .action-grid, .grid-2, .connect-grid { grid-template-columns: 1fr; } }
    @media (max-width: 640px) and (orientation: portrait) { .page { padding: 12px; } .hero, .section-header { flex-direction: column; } .hero h1 { font-size: 30px; } .hero p { font-size: 14px; } .grid-3, .role-grid, .player-grid { grid-template-columns: 1fr; } .vote-row { grid-template-columns: 1fr; } .tool-row input[type="range"] { width: 100%; } .btn { width: 100%; } .card { padding: 16px; border-radius: 22px; } }
    @media (max-width: 920px) and (orientation: landscape) { .page { padding: 10px; } .container { max-width: none; } .hero { align-items: center; margin-bottom: 10px; } .hero h1 { font-size: 24px; margin: 8px 0 0; } .hero p, .version { display: none; } .badge { padding: 6px 10px; font-size: 12px; } .hero-actions { flex-wrap: nowrap; } .hero-actions .btn, .btn { width: auto; min-height: 38px; padding: 8px 12px; border-radius: 14px; } .layout { grid-template-columns: minmax(0, 1.35fr) minmax(260px, .8fr); gap: 10px; align-items: start; } .main-column, .side-column { gap: 10px; } .card { padding: 12px; border-radius: 18px; } h2 { font-size: 18px; margin-bottom: 10px; } h3 { font-size: 15px; margin: 10px 0 8px; } p, small, .compact-note { font-size: 12px; } .grid-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 8px; } .grid-2, .connect-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 8px; } .form-grid { grid-template-columns: 1.2fr 1fr 1fr 1fr; gap: 8px; } .action-grid { grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 8px; margin: 8px 0; } .role-grid { grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 8px; } .player-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 8px; max-height: 46vh; overflow: auto; padding-right: 2px; } .stat-box, .box, .role-mini, .player-card, .log-list div { padding: 10px; border-radius: 16px; } .stat-box strong { font-size: 20px; margin: 4px 0; } input, select { padding: 9px 10px; border-radius: 12px; } .tool-row { margin: 8px 0; } .tool-row input[type="range"] { width: 140px; } .chat-box { height: 120px; padding: 8px; border-radius: 14px; } .log-list { max-height: 125px; overflow: auto; } .side-column .card:nth-child(2) { display: none; } .checkbox-line { padding: 10px; margin: 8px 0; border-radius: 14px; } .role-box { padding: 8px; } .role-box p { display: none; } }
  </style>
</head>
<body>
  <main class="page">
    <div class="container">
      <header class="hero">
        <div>
          <div class="badge">🌙 Ma Sói P2P Online</div>
          <h1>Ma Sói chơi chung bằng mã phòng</h1>
          <p>Một máy làm chủ phòng, máy khác nhập mã để kết nối P2P. Bot tự chơi độc lập theo timer.</p>
          <div class="version">Phiên bản v1.3.1</div>
        </div>
        <div class="hero-actions">
          <button class="btn secondary" id="resetBtn">🔄 Phòng mới</button>
          <button class="btn" id="copyBtn">📋 Copy mã</button>
        </div>
      </header>
      <div class="layout">
        <div class="main-column">
          <section class="card">
            <div class="grid-3">
              <div class="stat-box"><span>📡 Mã phòng</span><strong id="roomCodeText">----</strong><small id="networkHint">Bấm “Tạo phòng” để lấy mã thật.</small></div>
              <div class="stat-box"><span>👥 Người chơi</span><strong id="playerCountText">0/8</strong><small id="botCountText">Bot: 0</small></div>
              <div class="stat-box"><span>☀️ Trạng thái</span><strong id="phaseText">Sảnh</strong><small id="networkStatusText">Offline</small><small id="statusText">Game tiếp tục cho đến khi Làng hoặc Sói thắng.</small></div>
            </div>
          </section>
          <section class="card">
            <div class="section-header"><div><h2>🌐 Phòng online</h2><p>Tạo phòng hoặc nhập mã để vào phòng người khác.</p></div></div>
            <div class="connect-grid">
              <div class="box"><h3>Chủ phòng</h3><input id="hostNameInput" value="Chủ phòng" placeholder="Tên chủ phòng" /><div class="tool-row"><button class="btn success" id="createOnlineRoomBtn">🏠 Tạo phòng</button></div></div>
              <div class="box"><h3>Vào phòng</h3><input id="guestNameInput" placeholder="Tên của bạn" /><input id="joinRoomInput" placeholder="Nhập mã phòng" style="margin-top: 10px;" /><div class="tool-row"><button class="btn blue" id="joinOnlineRoomBtn">🚪 Vào phòng</button></div></div>
            </div>
            <div class="compact-note" id="peerStatusText">P2P dùng PeerJS để hai máy tìm nhau, không dùng Firebase/Supabase.</div>
          </section>
          <section class="card" id="lobbyCard">
            <div class="section-header"><div><h2>➕ Sảnh chờ</h2><p>Chủ phòng thêm người, thêm bot và bắt đầu ván.</p></div><button class="btn success" id="startBtn">▶️ Bắt đầu</button></div>
            <div class="form-grid"><input id="localAddNameInput" placeholder="Tên người chơi thêm thủ công" /><button class="btn secondary" id="addLocalHumanBtn">Thêm người</button><button class="btn ghost" id="addBotBtn">🤖 Thêm bot</button><button class="btn ghost" id="fillBotsBtn">Đủ bot</button></div>
            <div class="tool-row"><label for="minPlayersInput">Số người:</label><input id="minPlayersInput" type="range" min="5" max="12" value="8" /><b id="minPlayersText">8</b></div>
            <h3>✨ Vai trò khi đủ <span id="roleTargetText">8</span> người</h3><div class="role-grid" id="roleGrid"></div>
          </section>
          <section class="card hidden" id="gameCard">
            <div id="winnerBox" class="winner-box hidden"></div>
            <div id="nightPanel" class="hidden"><h2>🌙 Pha đêm</h2><div class="compact-note">Bot tự hành động sau vài giây. Chủ phòng chỉ chốt lượt.</div><div class="action-grid"><div class="box"><b>💀 Sói cắn</b><select id="wolfTargetSelect"></select></div><div class="box"><b>🛡️ Bảo vệ</b><select id="doctorTargetSelect"></select></div><div class="box"><b>👁️ Tiên tri</b><select id="seerTargetSelect"></select></div></div><label class="checkbox-line"><input type="checkbox" id="witchSaveInput" />💗 Phù thủy cứu</label><div class="tool-row"><button class="btn" id="resolveNightBtn">☀️ Kết thúc đêm</button></div></div>
            <div id="dayPanel" class="hidden"><h2>☀️ Pha ngày</h2><div class="grid-2"><div class="box"><b>Thông báo</b><p id="nightVictimText">Mọi người thức dậy và bắt đầu thảo luận.</p><p id="seerResultText" class="seer-result"></p></div><div class="box"><b>Bỏ phiếu</b><p>Người thật tự chọn. Bot tự vote sau vài giây.</p></div></div><div class="vote-list" id="voteList"></div><div class="tool-row"><button class="btn danger" id="resolveVoteBtn">💀 Chốt phiếu</button></div></div>
          </section>
          <section class="card"><h2>👥 Người chơi</h2><div class="player-grid" id="playerGrid"></div></section>
        </div>
        <aside class="side-column">
          <section class="card"><h2>💬 Chat phòng</h2><div class="chat-box" id="chatBox"></div><div class="chat-input"><input id="chatInput" placeholder="Nhập tin nhắn..." /><button class="btn" id="sendChatBtn">Gửi</button></div></section>
          <section class="card"><h2>👑 Luật thắng</h2><p><b>Làng:</b> loại hết Ma Sói.</p><p><b>Sói:</b> số sói còn sống bằng hoặc nhiều hơn dân làng.</p><p><b>Bot:</b> tự hành động, không nghe điều khiển người chơi.</p></section>
          <section class="card"><h2>📜 Nhật ký</h2><div class="log-list" id="logList"></div></section>
        </aside>
      </div>
    </div>
  </main>
  <script src="https://unpkg.com/peerjs@1.5.5/dist/peerjs.min.js"></script>
  <script>
    "use strict";
    const VERSION = "v1.3.1";
    const PEER_PREFIX = "ma-soi-p2p-";
    const BOT_THINK_MS = 2200;
    const ROLE_DATA = {
      wolf: { name: "Ma sói", team: "Sói", icon: "🐺", desc: "Mỗi đêm cùng bầy sói chọn một người để cắn." },
      villager: { name: "Dân làng", team: "Làng", icon: "🧑‍🌾", desc: "Không có kỹ năng ban đêm, thắng bằng suy luận và bỏ phiếu đúng." },
      seer: { name: "Tiên tri", team: "Làng", icon: "🔮", desc: "Mỗi đêm soi một người để biết họ thuộc phe Sói hay không." },
      doctor: { name: "Bảo vệ", team: "Làng", icon: "🛡️", desc: "Mỗi đêm bảo vệ một người khỏi bị sói cắn." },
      witch: { name: "Phù thủy", team: "Làng", icon: "🧪", desc: "Có một bình cứu. Bản đơn giản cho phép cứu nạn nhân đêm đó." },
      hunter: { name: "Thợ săn", team: "Làng", icon: "🏹", desc: "Vai mạnh trong làng. Bản này hiển thị vai trước, chưa xử lý bắn khi chết." },
      cupid: { name: "Thần tình yêu", team: "Làng", icon: "💘", desc: "Vai đặc biệt. Bản này hiển thị vai trước, chưa xử lý ghép đôi." },
      guard: { name: "Hiệp sĩ", team: "Làng", icon: "⚔️", desc: "Có tiếng nói mạnh khi tranh luận. Bản này tính phiếu như người thường." },
      alphaWolf: { name: "Sói đầu đàn", team: "Sói", icon: "👑🐺", desc: "Sói đặc biệt, cùng phe Sói đi cắn mỗi đêm." }
    };
    const BOT_NAMES = ["An Bot", "Bình Bot", "Chi Bot", "Dũng Bot", "Em Bot", "Giang Bot", "Hạ Bot", "Khoa Bot", "Linh Bot", "Minh Bot", "Nhi Bot", "Phong Bot", "Quân Bot", "Trang Bot", "Vy Bot", "Long Bot", "Tú Bot", "Huy Bot"];
    const ROLE_SET_BY_SIZE = {
      5: ["wolf", "seer", "doctor", "villager", "villager"], 6: ["wolf", "seer", "doctor", "hunter", "villager", "villager"], 7: ["wolf", "wolf", "seer", "doctor", "hunter", "villager", "villager"], 8: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "villager", "villager"], 9: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"], 10: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"], 11: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager"], 12: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager", "villager"]
    };
    const state = { roomCode: makeCode(), localName: "Chủ phòng", localPlayerId: "", minPlayers: 8, players: [], phase: "lobby", day: 0, selectedWolfTarget: "", selectedDoctorTarget: "", selectedSeerTarget: "", selectedWitchSave: false, seerResult: "", nightVictim: "", votes: {}, log: ["Game đã mở. Tạo phòng online hoặc vào phòng bằng mã."], chat: [{ from: "Hệ thống", text: "Chào mừng đến với Ma Sói P2P!" }], botTimer: null, network: { mode: "offline", status: "Offline", peerReady: false, hostPeerId: "", connections: {} } };
    let peer = null;
    let hostConnection = null;
    const els = {};
    document.addEventListener("DOMContentLoaded", init);
    function init() { cacheElements(); const ok = runSelfTests(); if (!ok) return; bindEvents(); setupOfflineHost(); render(); }
    function cacheElements() { ["resetBtn","copyBtn","roomCodeText","networkHint","playerCountText","networkStatusText","statusText","peerStatusText","botCountText","hostNameInput","guestNameInput","joinRoomInput","createOnlineRoomBtn","joinOnlineRoomBtn","phaseText","lobbyCard","gameCard","winnerBox","nightPanel","dayPanel","localAddNameInput","addLocalHumanBtn","addBotBtn","fillBotsBtn","startBtn","minPlayersInput","minPlayersText","roleTargetText","roleGrid","wolfTargetSelect","doctorTargetSelect","seerTargetSelect","witchSaveInput","resolveNightBtn","nightVictimText","seerResultText","voteList","resolveVoteBtn","playerGrid","chatBox","chatInput","sendChatBtn","logList"].forEach((id) => { els[id] = document.getElementById(id); }); }
    function mustEl(name) { return els[name] || null; }
    function setText(name, value) { const el = mustEl(name); if (el) el.textContent = value; }
    function setHtml(name, value) { const el = mustEl(name); if (el) el.innerHTML = value; }
    function bindEvents() { on("resetBtn","click",resetGame); on("copyBtn","click",copyInvite); on("createOnlineRoomBtn","click",createOnlineRoom); on("joinOnlineRoomBtn","click",joinOnlineRoom); on("addLocalHumanBtn","click",addLocalHuman); on("addBotBtn","click",() => addBot(1)); on("fillBotsBtn","click",fillBots); on("startBtn","click",startGame); on("resolveNightBtn","click",resolveNight); on("resolveVoteBtn","click",resolveVote); on("sendChatBtn","click",sendChat); on("chatInput","keydown",(event) => { if (event.key === "Enter") sendChat(); }); on("minPlayersInput","input",() => { if (!isHost()) return; state.minPlayers = Number(els.minPlayersInput.value); syncAfterHostChange(); }); on("witchSaveInput","change",() => { if (!isHost()) return; state.selectedWitchSave = els.witchSaveInput.checked; syncAfterHostChange(); }); on("wolfTargetSelect","change",() => { if (!isHost()) return; state.selectedWolfTarget = els.wolfTargetSelect.value; syncAfterHostChange(); }); on("doctorTargetSelect","change",() => { if (!isHost()) return; state.selectedDoctorTarget = els.doctorTargetSelect.value; syncAfterHostChange(); }); on("seerTargetSelect","change",() => { if (!isHost()) return; state.selectedSeerTarget = els.seerTargetSelect.value; syncAfterHostChange(); }); }
    function on(id, eventName, handler) { const el = els[id]; if (el) el.addEventListener(eventName, handler); }
    function setupOfflineHost() { const host = createPlayer("Chủ phòng", false, true, "local"); state.localPlayerId = host.id; state.players = [host]; }
    function makeCode() { return Math.random().toString(36).slice(2, 6).toUpperCase() + "-" + Math.random().toString(36).slice(2, 6).toUpperCase(); }
    function makePeerIdFromCode(code) { return PEER_PREFIX + String(code || "").toLowerCase().replace(/[^a-z0-9-]/g, ""); }
    function makeId() { return window.crypto && typeof window.crypto.randomUUID === "function" ? window.crypto.randomUUID() : "id-" + Date.now() + "-" + Math.random().toString(36).slice(2); }
    function createPlayer(name, isBot, host, connectionId) { return { id: makeId(), name, isBot: Boolean(isBot), host: Boolean(host), connectionId: connectionId || "", alive: true, role: null }; }
    function shuffle(arr) { return arr.slice().sort(() => Math.random() - 0.5); }
    function chooseRandom(list) { return Array.isArray(list) && list.length ? list[Math.floor(Math.random() * list.length)] : null; }
    function getRoleSet(count) { const safeCount = Math.max(5, Math.min(12, Number(count) || 5)); return ROLE_SET_BY_SIZE[safeCount] || ROLE_SET_BY_SIZE[12]; }
    function isWolfRole(role) { return role === "wolf" || role === "alphaWolf"; }
    function alivePlayers() { return state.players.filter((player) => player.alive); }
    function wolvesAlive() { return state.players.filter((player) => player.alive && isWolfRole(player.role)); }
    function villagersAlive() { return state.players.filter((player) => player.alive && !isWolfRole(player.role)); }
    function isHost() { return state.network.mode === "host" || state.network.mode === "offline"; }
    function getWinner() { if (state.phase === "lobby") return ""; if (wolvesAlive().length === 0) return "Làng thắng! Tất cả ma sói đã bị loại."; if (wolvesAlive().length >= villagersAlive().length) return "Sói thắng! Số sói đã bằng hoặc vượt số dân làng."; return ""; }
    function syncAfterHostChange() { scheduleBotBrain(); broadcastState(); render(); }
    function pushLog(text, shouldBroadcast) { state.log.unshift(text); state.log = state.log.slice(0, 16); if (shouldBroadcast !== false) broadcastState(); render(); }
    function resetGame() { clearBotTimer(); closeNetwork(); state.roomCode = makeCode(); state.localName = "Chủ phòng"; state.minPlayers = 8; state.phase = "lobby"; state.day = 0; state.selectedWolfTarget = ""; state.selectedDoctorTarget = ""; state.selectedSeerTarget = ""; state.selectedWitchSave = false; state.seerResult = ""; state.nightVictim = ""; state.votes = {}; state.log = ["Phòng mới đã được tạo. Tạo phòng online hoặc chơi offline."]; state.chat = [{ from: "Hệ thống", text: "Phòng mới đã sẵn sàng." }]; setupOfflineHost(); render(); }
    function closeNetwork() { Object.values(state.network.connections).forEach((conn) => { try { conn.close(); } catch (error) {} }); state.network.connections = {}; if (hostConnection) { try { hostConnection.close(); } catch (error) {} hostConnection = null; } if (peer) { try { peer.destroy(); } catch (error) {} peer = null; } state.network.mode = "offline"; state.network.status = "Offline"; state.network.peerReady = false; state.network.hostPeerId = ""; }
    async function copyInvite() { const invite = "Mời bạn vào chơi Ma Sói P2P! Mã phòng: " + state.roomCode + ". Mở link game, nhập tên và nhập mã này để vào phòng."; try { if (navigator.clipboard && navigator.clipboard.writeText) { await navigator.clipboard.writeText(invite); setText("copyBtn", "✅ Đã copy"); setTimeout(() => render(), 1200); } else pushLog("Trình duyệt không hỗ trợ copy tự động. Mã phòng là " + state.roomCode + ".", false); } catch (error) { pushLog("Không copy tự động được. Mã phòng là " + state.roomCode + ".", false); } }
    function requirePeerLibrary() { if (typeof Peer === "undefined") { pushLog("Không tải được PeerJS. Hãy kiểm tra internet/adblock rồi tải lại trang.", false); return false; } return true; }
    function createOnlineRoom() { if (!requirePeerLibrary()) return; clearBotTimer(); closeNetwork(); const hostName = cleanName(els.hostNameInput.value || "Chủ phòng"); state.roomCode = makeCode(); const hostPlayer = createPlayer(hostName, false, true, "host"); state.localPlayerId = hostPlayer.id; state.localName = hostName; state.players = [hostPlayer]; state.phase = "lobby"; state.day = 0; state.votes = {}; state.network.mode = "host"; state.network.status = "Đang tạo phòng"; state.network.hostPeerId = makePeerIdFromCode(state.roomCode); render(); peer = new Peer(state.network.hostPeerId, { debug: 1 }); peer.on("open", () => { state.network.peerReady = true; state.network.status = "Đang mở phòng"; pushLog("Phòng online đã mở. Mã phòng: " + state.roomCode + ".", false); render(); }); peer.on("connection", setupHostConnection); peer.on("error", (error) => { state.network.status = "Lỗi"; pushLog("Lỗi tạo phòng online: " + readablePeerError(error), false); render(); }); }
    function joinOnlineRoom() { if (!requirePeerLibrary()) return; const guestName = cleanName(els.guestNameInput.value); const code = els.joinRoomInput.value.trim().toUpperCase(); if (!guestName) { pushLog("Hãy nhập tên của bạn trước khi vào phòng.", false); return; } if (!code) { pushLog("Hãy nhập mã phòng trước khi vào.", false); return; } clearBotTimer(); closeNetwork(); state.roomCode = code; state.localName = guestName; state.network.mode = "guest"; state.network.status = "Đang vào phòng"; state.network.hostPeerId = makePeerIdFromCode(code); state.players = []; render(); peer = new Peer(undefined, { debug: 1 }); peer.on("open", () => { state.network.peerReady = true; hostConnection = peer.connect(state.network.hostPeerId, { reliable: true }); setupGuestConnection(hostConnection, guestName); }); peer.on("error", (error) => { state.network.status = "Lỗi"; pushLog("Lỗi vào phòng: " + readablePeerError(error), false); render(); }); }
    function setupHostConnection(conn) { state.network.connections[conn.peer] = conn; conn.on("open", () => { sendTo(conn, { type: "hello-host" }); broadcastState(); render(); }); conn.on("data", (data) => handleHostData(conn, safeMessage(data))); conn.on("close", () => { delete state.network.connections[conn.peer]; const player = state.players.find((item) => item.connectionId === conn.peer); if (player) pushLog(player.name + " đã mất kết nối.", false); broadcastState(); render(); }); conn.on("error", () => { delete state.network.connections[conn.peer]; pushLog("Một kết nối bị lỗi.", false); render(); }); }
    function setupGuestConnection(conn, guestName) { conn.on("open", () => { state.network.status = "Đã kết nối"; sendTo(conn, { type: "join", name: guestName }); pushLog("Đã gửi yêu cầu vào phòng.", false); render(); }); conn.on("data", (data) => handleGuestData(safeMessage(data))); conn.on("close", () => { state.network.status = "Mất kết nối"; pushLog("Đã mất kết nối với chủ phòng.", false); render(); }); conn.on("error", () => { state.network.status = "Lỗi kết nối"; pushLog("Kết nối tới chủ phòng bị lỗi.", false); render(); }); }
    function handleHostData(conn, message) { if (!message || !message.type) return; if (message.type === "join") { if (state.players.length >= 12) { sendTo(conn, { type: "reject", reason: "Phòng đã đủ 12 người." }); return; } let player = state.players.find((item) => item.connectionId === conn.peer); if (!player) { player = createPlayer(cleanName(message.name || "Người chơi"), false, false, conn.peer); state.players.push(player); pushLog(player.name + " đã vào phòng online.", false); } sendTo(conn, { type: "joined", state: publicState(), playerId: player.id }); broadcastState(); render(); return; } if (message.type === "chat") { addChat(cleanName(message.from || "Người chơi"), String(message.text || ""), false); broadcastState(); render(); return; } if (message.type === "vote" && state.phase === "day") { state.votes[message.voterId] = message.targetId; broadcastState(); render(); } }
    function handleGuestData(message) { if (!message || !message.type) return; if (message.type === "state" || message.type === "joined") { if (message.playerId) state.localPlayerId = message.playerId; applyPublicState(message.state); state.network.status = "Đã kết nối"; render(); return; } if (message.type === "reject") { state.network.status = "Bị từ chối"; pushLog(message.reason || "Không thể vào phòng.", false); render(); } }
    function publicState() { return { roomCode: state.roomCode, minPlayers: state.minPlayers, players: state.players, phase: state.phase, day: state.day, selectedWolfTarget: state.selectedWolfTarget, selectedDoctorTarget: state.selectedDoctorTarget, selectedSeerTarget: state.selectedSeerTarget, selectedWitchSave: state.selectedWitchSave, seerResult: state.seerResult, nightVictim: state.nightVictim, votes: state.votes, log: state.log, chat: state.chat }; }
    function applyPublicState(next) { if (!next) return; state.roomCode = next.roomCode || state.roomCode; state.minPlayers = Number(next.minPlayers) || state.minPlayers; state.players = Array.isArray(next.players) ? next.players : state.players; state.phase = next.phase || state.phase; state.day = Number(next.day) || 0; state.selectedWolfTarget = next.selectedWolfTarget || ""; state.selectedDoctorTarget = next.selectedDoctorTarget || ""; state.selectedSeerTarget = next.selectedSeerTarget || ""; state.selectedWitchSave = Boolean(next.selectedWitchSave); state.seerResult = next.seerResult || ""; state.nightVictim = next.nightVictim || ""; state.votes = next.votes || {}; state.log = Array.isArray(next.log) ? next.log : state.log; state.chat = Array.isArray(next.chat) ? next.chat : state.chat; }
    function broadcastState() { if (state.network.mode !== "host") return; const msg = { type: "state", state: publicState() }; Object.values(state.network.connections).forEach((conn) => sendTo(conn, msg)); }
    function sendTo(conn, message) { try { if (conn && conn.open) conn.send(message); } catch (error) { pushLog("Gửi dữ liệu thất bại.", false); } }
    function safeMessage(data) { return data && typeof data === "object" ? data : null; }
    function readablePeerError(error) { const type = error && error.type ? error.type : "unknown"; if (type === "unavailable-id") return "Mã phòng này đang được dùng. Hãy bấm tạo phòng mới."; if (type === "peer-unavailable") return "Không tìm thấy phòng. Kiểm tra mã hoặc chủ phòng đã tắt."; if (type === "network") return "Lỗi mạng hoặc server tín hiệu không phản hồi."; return type; }
    function cleanName(value) { return String(value || "").trim().slice(0, 24) || "Người chơi"; }
    function addLocalHuman() { if (!isHost()) return; if (state.players.length >= 12) { pushLog("Phòng đã đủ 12 người."); return; } const name = cleanName(els.localAddNameInput.value); state.players.push(createPlayer(name, false, false, "manual")); els.localAddNameInput.value = ""; pushLog(name + " đã được thêm thủ công."); scheduleBotBrain(); }
    function addBot(count) { if (!isHost()) return; if (state.players.length >= 12) { pushLog("Phòng đã đủ 12 người, không thể thêm bot."); return; } const amount = Math.min(Number(count) || 1, 12 - state.players.length); const used = new Set(state.players.map((player) => player.name)); for (let i = 0; i < amount; i++) { const botName = BOT_NAMES.find((name) => !used.has(name)) || "Bot " + (state.players.length + 1); used.add(botName); state.players.push(createPlayer(botName, true, false, "bot")); } pushLog("Đã thêm " + amount + " bot. Bot sẽ tự chơi độc lập."); scheduleBotBrain(); }
    function fillBots() { if (!isHost()) return; const need = Math.max(0, state.minPlayers - state.players.length); if (need > 0) addBot(need); else pushLog("Số người đã đủ, không cần thêm bot."); }
    function startGame() { if (!isHost()) return; let roster = state.players.slice(); const need = Math.max(0, state.minPlayers - roster.length); for (let i = 0; i < need; i++) { const botName = BOT_NAMES.find((name) => !roster.some((player) => player.name === name)) || "Bot " + (roster.length + 1); roster.push(createPlayer(botName, true, false, "bot")); } roster = roster.slice(0, 12); const roles = shuffle(getRoleSet(roster.length)); state.players = shuffle(roster).map((player, index) => ({ ...player, role: roles[index], alive: true })); state.phase = "night"; state.day = 1; state.votes = {}; state.nightVictim = ""; state.seerResult = ""; state.selectedWolfTarget = ""; state.selectedDoctorTarget = ""; state.selectedSeerTarget = ""; state.selectedWitchSave = false; pushLog("Game bắt đầu với " + state.players.length + " người. Bot sẽ tự hành động trong đêm."); scheduleBotBrain(); }
    function clearBotTimer() { if (state.botTimer) { clearTimeout(state.botTimer); state.botTimer = null; } }
    function scheduleBotBrain() { clearBotTimer(); if (!isHost() || state.phase === "lobby" || getWinner()) return; state.botTimer = setTimeout(() => { state.botTimer = null; runBotBrain(); }, BOT_THINK_MS); }
    function runBotBrain() { if (!isHost() || state.phase === "lobby" || getWinner()) return; let changed = false; if (state.phase === "night") changed = botNightActions(); if (state.phase === "day") changed = botDayVotes(); if (changed) { state.log.unshift("Bot đã tự hành động độc lập."); state.log = state.log.slice(0, 16); broadcastState(); render(); } }
    function botNightActions() { const alive = alivePlayers(); const botWolves = alive.filter((p) => p.isBot && isWolfRole(p.role)); const humanWolves = alive.filter((p) => !p.isBot && isWolfRole(p.role)); const botDoctors = alive.filter((p) => p.isBot && p.role === "doctor"); const botSeers = alive.filter((p) => p.isBot && p.role === "seer"); const botWitches = alive.filter((p) => p.isBot && p.role === "witch"); let changed = false; if (!state.selectedWolfTarget && botWolves.length > 0 && humanWolves.length === 0) { const victim = chooseRandom(alive.filter((p) => !isWolfRole(p.role))); if (victim) { state.selectedWolfTarget = victim.id; changed = true; } } if (!state.selectedDoctorTarget && botDoctors.length > 0) { const protect = chooseRandom(alive.filter((p) => !isWolfRole(p.role))) || chooseRandom(alive); if (protect) { state.selectedDoctorTarget = protect.id; changed = true; } } if (!state.selectedSeerTarget && botSeers.length > 0) { const target = chooseRandom(alive.filter((p) => p.role !== "seer")); if (target) { state.selectedSeerTarget = target.id; changed = true; } } if (!state.selectedWitchSave && botWitches.length > 0 && state.selectedWolfTarget && Math.random() < 0.45) { state.selectedWitchSave = true; changed = true; } return changed; }
    function botDayVotes() { const alive = alivePlayers(); const bots = alive.filter((p) => p.isBot); let changed = false; bots.forEach((bot) => { if (state.votes[bot.id]) return; const candidates = alive.filter((p) => p.id !== bot.id); let pool = candidates; if (!isWolfRole(bot.role)) { const wolves = candidates.filter((p) => isWolfRole(p.role)); if (wolves.length && Math.random() < 0.55) pool = wolves; } else { const nonWolves = candidates.filter((p) => !isWolfRole(p.role)); if (nonWolves.length) pool = nonWolves; } const pick = chooseRandom(pool); if (pick) { state.votes[bot.id] = pick.id; changed = true; } }); return changed; }
    function resolveNight() { if (!isHost()) return; botNightActions(); const wolfTarget = state.players.find((player) => player.id === state.selectedWolfTarget); const doctorTarget = state.players.find((player) => player.id === state.selectedDoctorTarget); const seerTarget = state.players.find((player) => player.id === state.selectedSeerTarget); let victim = null; if (wolfTarget) { const savedByDoctor = doctorTarget && doctorTarget.id === wolfTarget.id; const savedByWitch = state.selectedWitchSave; if (!savedByDoctor && !savedByWitch) { victim = wolfTarget; state.players = state.players.map((player) => player.id === wolfTarget.id ? { ...player, alive: false } : player); } } state.seerResult = seerTarget ? seerTarget.name + " " + (isWolfRole(seerTarget.role) ? "thuộc phe Sói" : "không thuộc phe Sói") + "." : ""; state.nightVictim = victim ? victim.name + " đã bị sói cắn." : "Không ai chết trong đêm nay."; state.phase = "day"; state.votes = {}; pushLog(victim ? "Sáng ngày " + state.day + ": " + victim.name + " đã chết." : "Sáng ngày " + state.day + ": không ai chết."); scheduleBotBrain(); }
    function resolveVote() { if (!isHost()) return; botDayVotes(); const count = {}; Object.values(state.votes).forEach((targetId) => { if (targetId) count[targetId] = (count[targetId] || 0) + 1; }); const sorted = Object.entries(count).sort((a, b) => b[1] - a[1]); if (!sorted.length) { state.phase = "night"; state.day += 1; pushLog("Không đủ phiếu, không ai bị treo cổ. Đêm tiếp theo bắt đầu."); scheduleBotBrain(); return; } const targetId = sorted[0][0]; const amount = sorted[0][1]; const eliminated = state.players.find((player) => player.id === targetId); state.players = state.players.map((player) => player.id === targetId ? { ...player, alive: false } : player); state.phase = "night"; state.day += 1; state.selectedWolfTarget = ""; state.selectedDoctorTarget = ""; state.selectedSeerTarget = ""; state.selectedWitchSave = false; state.nightVictim = ""; state.votes = {}; pushLog((eliminated ? eliminated.name : "Một người") + " bị treo cổ với " + amount + " phiếu. Đêm tiếp theo bắt đầu."); scheduleBotBrain(); }
    function sendChat() { const text = els.chatInput.value.trim(); if (!text) return; const from = state.localName || "Người chơi"; if (state.network.mode === "guest" && hostConnection && hostConnection.open) sendTo(hostConnection, { type: "chat", from, text }); else addChat(from, text, true); els.chatInput.value = ""; render(); }
    function addChat(from, text, shouldBroadcast) { state.chat.push({ from: cleanName(from), text: String(text).slice(0, 200) }); state.chat = state.chat.slice(-10); if (shouldBroadcast !== false) broadcastState(); }
    function castVote(voterId, targetId) { if (state.network.mode === "guest") sendTo(hostConnection, { type: "vote", voterId, targetId }); else if (isHost()) { state.votes[voterId] = targetId; syncAfterHostChange(); } }
    function render() { const winner = getWinner(); const bots = state.players.filter((player) => player.isBot).length; setText("roomCodeText", state.roomCode); setText("playerCountText", state.players.length + "/" + state.minPlayers); setText("networkStatusText", state.network.status); setText("peerStatusText", state.network.mode === "host" ? "Bạn là chủ phòng. Gửi mã cho bạn bè." : state.network.mode === "guest" ? "Bạn đang nhận dữ liệu từ chủ phòng." : "P2P dùng PeerJS để hai máy tìm nhau, không dùng Firebase/Supabase."); setText("networkHint", state.network.mode === "host" ? "Mã này cho người khác nhập để vào phòng." : state.network.mode === "guest" ? "Bạn đang ở phòng " + state.roomCode + "." : "Bấm “Tạo phòng” để lấy mã thật."); setText("botCountText", "Bot: " + bots); setText("phaseText", state.phase === "lobby" ? "Sảnh" : state.phase === "night" ? "Đêm " + state.day : "Ngày " + state.day); setText("statusText", winner || "Game tiếp tục cho đến khi Làng hoặc Sói thắng."); if (els.minPlayersInput) els.minPlayersInput.value = String(state.minPlayers); setText("minPlayersText", String(state.minPlayers)); setText("roleTargetText", String(state.minPlayers)); if (els.witchSaveInput) els.witchSaveInput.checked = state.selectedWitchSave; setControlsEnabled(); toggle(els.lobbyCard, state.phase !== "lobby"); toggle(els.gameCard, state.phase === "lobby"); toggle(els.winnerBox, !winner); toggle(els.nightPanel, state.phase !== "night" || Boolean(winner)); toggle(els.dayPanel, state.phase !== "day" || Boolean(winner)); if (winner && els.winnerBox) els.winnerBox.innerHTML = "<h2>" + escapeHtml(winner) + "</h2><p>Bấm “Phòng mới” để chơi ván khác.</p>"; renderRoles(); renderNightSelects(); renderVotes(); renderPlayers(); renderChat(); renderLog(); setText("nightVictimText", state.nightVictim || "Mọi người thức dậy và bắt đầu thảo luận."); setText("seerResultText", state.seerResult ? "Kết quả riêng của Tiên tri: " + state.seerResult : ""); }
    function setControlsEnabled() { [els.addLocalHumanBtn, els.addBotBtn, els.fillBotsBtn, els.startBtn, els.minPlayersInput, els.wolfTargetSelect, els.doctorTargetSelect, els.seerTargetSelect, els.witchSaveInput, els.resolveNightBtn, els.resolveVoteBtn].forEach((el) => { if (el) el.disabled = !isHost(); }); if (els.createOnlineRoomBtn) els.createOnlineRoomBtn.disabled = state.network.mode === "host" && state.network.peerReady; if (els.joinOnlineRoomBtn) els.joinOnlineRoomBtn.disabled = state.network.mode === "guest" && state.network.status === "Đã kết nối"; }
    function renderRoles() { const counts = {}; getRoleSet(state.minPlayers).forEach((role) => { counts[role] = (counts[role] || 0) + 1; }); setHtml("roleGrid", Object.keys(counts).map((role) => { const data = ROLE_DATA[role]; return "<div class=\"role-mini\"><span>" + data.icon + "</span><b>" + escapeHtml(data.name) + " x" + counts[role] + "</b><small>Phe " + escapeHtml(data.team) + "</small></div>"; }).join("")); }
    function renderNightSelects() { const alive = alivePlayers(); fillSelect(els.wolfTargetSelect, alive.filter((player) => !isWolfRole(player.role)), state.selectedWolfTarget, "Chưa chọn"); fillSelect(els.doctorTargetSelect, alive, state.selectedDoctorTarget, "Chưa chọn"); fillSelect(els.seerTargetSelect, alive, state.selectedSeerTarget, "Chưa chọn"); }
    function fillSelect(selectEl, players, selectedValue, emptyLabel) { if (!selectEl) return; selectEl.innerHTML = "<option value=\"\">" + escapeHtml(emptyLabel) + "</option>"; players.forEach((player) => { const option = document.createElement("option"); option.value = player.id; option.textContent = player.name; selectEl.appendChild(option); }); selectEl.value = selectedValue; }
    function renderVotes() { const alive = alivePlayers(); if (!els.voteList) return; els.voteList.innerHTML = ""; alive.forEach((voter) => { const row = document.createElement("div"); row.className = "vote-row"; const label = document.createElement("b"); label.textContent = voter.name + (voter.isBot ? " BOT" : ""); const select = document.createElement("select"); select.innerHTML = "<option value=\"\">Chọn người bị nghi</option>"; alive.filter((player) => player.id !== voter.id).forEach((candidate) => { const option = document.createElement("option"); option.value = candidate.id; option.textContent = candidate.name; select.appendChild(option); }); select.value = state.votes[voter.id] || ""; select.disabled = voter.isBot || (!isHost() && voter.id !== state.localPlayerId); select.addEventListener("change", () => castVote(voter.id, select.value)); row.appendChild(label); row.appendChild(select); els.voteList.appendChild(row); }); }
    function renderPlayers() { setHtml("playerGrid", state.players.map((player) => { const role = player.role ? ROLE_DATA[player.role] : null; const status = (player.host ? "Chủ phòng " : "") + (player.isBot ? "BOT tự chơi " : "") + (player.id === state.localPlayerId ? "Bạn " : "") + (player.alive ? "Còn sống" : "Đã chết"); const roleBox = role ? "<div class=\"role-box\"><b>" + escapeHtml(role.name) + "</b><small>Phe " + escapeHtml(role.team) + "</small><p>" + escapeHtml(role.desc) + "</p></div>" : ""; return "<div class=\"player-card " + (player.alive ? "" : "dead") + "\"><div class=\"player-top\"><div><b>" + escapeHtml(player.name) + "</b><small>" + escapeHtml(status) + "</small></div><span>" + (role ? role.icon : player.isBot ? "🤖" : "🙂") + "</span></div>" + roleBox + "</div>"; }).join("")); }
    function renderChat() { if (!els.chatBox) return; els.chatBox.innerHTML = state.chat.map((m) => "<p><b>" + escapeHtml(m.from) + ":</b> " + escapeHtml(m.text) + "</p>").join(""); els.chatBox.scrollTop = els.chatBox.scrollHeight; }
    function renderLog() { setHtml("logList", state.log.map((entry) => "<div>" + escapeHtml(entry) + "</div>").join("")); }
    function toggle(el, shouldHide) { if (el) el.classList.toggle("hidden", shouldHide); }
    function escapeHtml(value) { return String(value).replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/\"/g, "&quot;").replace(/'/g, "&#039;"); }
    function runSelfTests() { const required = ["resetBtn","copyBtn","roomCodeText","networkHint","playerCountText","networkStatusText","statusText","peerStatusText","botCountText","hostNameInput","guestNameInput","joinRoomInput","createOnlineRoomBtn","joinOnlineRoomBtn","phaseText","lobbyCard","gameCard","winnerBox","nightPanel","dayPanel","localAddNameInput","addLocalHumanBtn","addBotBtn","fillBotsBtn","startBtn","minPlayersInput","minPlayersText","roleTargetText","roleGrid","wolfTargetSelect","doctorTargetSelect","seerTargetSelect","witchSaveInput","resolveNightBtn","nightVictimText","seerResultText","voteList","resolveVoteBtn","playerGrid","chatBox","chatInput","sendChatBtn","logList"]; const missing = required.filter((id) => !els[id]); const bot = createPlayer("Bot Test", true, false, "bot"); const tests = [ { name: "Không thiếu DOM ID", pass: missing.length === 0 }, { name: "Phiên bản đúng v1.3.1", pass: VERSION === "v1.3.1" }, { name: "Bộ 5 người có đúng 5 vai", pass: getRoleSet(5).length === 5 }, { name: "Bộ 8 người có Sói đầu đàn", pass: getRoleSet(8).includes("alphaWolf") }, { name: "Số quá lớn được giới hạn về 12 vai", pass: getRoleSet(99).length === 12 }, { name: "Chọn ngẫu nhiên danh sách rỗng trả về null", pass: chooseRandom([]) === null }, { name: "Bot được đánh dấu là người máy", pass: bot.isBot === true }, { name: "Bot có connectionId riêng là bot", pass: bot.connectionId === "bot" }, { name: "Hàm kiểm tra PeerJS tồn tại", pass: typeof requirePeerLibrary === "function" } ]; tests.forEach((test) => console.assert(test.pass, "Self-test failed: " + test.name)); if (missing.length) { document.body.innerHTML = "<pre style='color:#fca5a5;background:#020617;padding:20px;white-space:pre-wrap'>Lỗi DOM: thiếu ID sau:\n" + missing.join("\n") + "</pre>"; return false; } if (tests.every((test) => test.pass)) console.log("Ma Sói P2P " + VERSION + ": self-test passed"); return tests.every((test) => test.pass); }
  </script>
</body>
</html>
    input:focus, select:focus { border-color: #a78bfa; }
    .grid-3 { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }
    .grid-2 { display: grid; grid-template-columns: repeat(2, 1fr); gap: 14px; }
    .stat-box, .box, .role-mini, .player-card, .log-list div { background: var(--panel); border: 1px solid var(--line); border-radius: 22px; padding: 16px; }
    .stat-box span { display: block; color: var(--muted); font-size: 14px; }
    .stat-box strong { display: block; color: #d8b4fe; font-size: 30px; margin: 8px 0; word-break: break-word; }
    small { display: block; color: var(--muted); }
    .section-header { display: flex; justify-content: space-between; align-items: flex-start; gap: 16px; margin-bottom: 18px; }
    .section-header p { margin: 4px 0 0; color: var(--muted); }
    .form-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
    .connect-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    .tool-row { margin: 16px 0; }
    .tool-row input[type="range"] { width: 220px; padding: 0; }
    .role-grid, .player-grid, .action-grid { display: grid; gap: 10px; }
    .role-grid { grid-template-columns: repeat(4, 1fr); }
    .player-grid { grid-template-columns: repeat(3, 1fr); }
    .action-grid { grid-template-columns: repeat(3, 1fr); margin: 16px 0; }
    .role-mini span { font-size: 25px; }
    .role-mini b { display: block; margin-top: 6px; }
    .checkbox-line { display: flex; align-items: center; gap: 10px; background: var(--panel); border: 1px solid var(--line); border-radius: 20px; padding: 14px; margin: 14px 0; }
    .checkbox-line input { width: auto; }
    .winner-box { background: rgba(16,185,129,.14); border: 1px solid rgba(110,231,183,.3); border-radius: 24px; padding: 22px; }
    .winner-box h2 { color: #a7f3d0; font-size: 28px; }
    .seer-result { color: #d8b4fe; }
    .vote-list { display: flex; flex-direction: column; gap: 10px; }
    .vote-row { display: grid; grid-template-columns: 1fr 2fr; gap: 10px; align-items: center; background: var(--panel); border: 1px solid var(--line); border-radius: 18px; padding: 12px; }
    .player-card.dead { background: rgba(127,29,29,.26); border-color: rgba(248,113,113,.35); opacity: .74; }
    .player-top { display: flex; justify-content: space-between; gap: 12px; }
    .player-top b { font-size: 18px; }
    .player-top span { font-size: 28px; }
    .role-box { margin-top: 12px; background: #0f172a; border-radius: 16px; padding: 12px; }
    .role-box p { color: var(--muted); font-size: 13px; margin-bottom: 0; }
    .chat-box { height: 230px; overflow: auto; background: var(--panel); border: 1px solid var(--line); border-radius: 18px; padding: 12px; margin-bottom: 10px; }
    .chat-box p { margin: 0 0 8px; color: var(--soft); }
    .chat-box b { color: #d8b4fe; }
    .log-list { display: flex; flex-direction: column; gap: 10px; }
    .log-list div { color: var(--soft); font-size: 14px; }
    .compact-note { color: var(--muted); font-size: 13px; margin-top: 10px; }
    .hidden { display: none !important; }
    @media (max-width: 1050px) { .layout { grid-template-columns: 1fr; } .grid-3, .role-grid, .player-grid { grid-template-columns: repeat(2, 1fr); } .form-grid, .action-grid, .grid-2, .connect-grid { grid-template-columns: 1fr; } }
    @media (max-width: 640px) and (orientation: portrait) { .page { padding: 12px; } .hero, .section-header { flex-direction: column; } .hero h1 { font-size: 30px; } .hero p { font-size: 14px; } .grid-3, .role-grid, .player-grid { grid-template-columns: 1fr; } .vote-row { grid-template-columns: 1fr; } .tool-row input[type="range"] { width: 100%; } .btn { width: 100%; } .card { padding: 16px; border-radius: 22px; } }
    @media (max-width: 920px) and (orientation: landscape) { .page { padding: 10px; } .container { max-width: none; } .hero { align-items: center; margin-bottom: 10px; } .hero h1 { font-size: 24px; margin: 8px 0 0; } .hero p, .version { display: none; } .badge { padding: 6px 10px; font-size: 12px; } .hero-actions { flex-wrap: nowrap; } .hero-actions .btn, .btn { width: auto; min-height: 38px; padding: 8px 12px; border-radius: 14px; } .layout { grid-template-columns: minmax(0, 1.35fr) minmax(260px, .8fr); gap: 10px; align-items: start; } .main-column, .side-column { gap: 10px; } .card { padding: 12px; border-radius: 18px; } h2 { font-size: 18px; margin-bottom: 10px; } h3 { font-size: 15px; margin: 10px 0 8px; } p, small, .compact-note { font-size: 12px; } .grid-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 8px; } .grid-2, .connect-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 8px; } .form-grid { grid-template-columns: 1.2fr 1fr 1fr 1fr; gap: 8px; } .action-grid { grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 8px; margin: 8px 0; } .role-grid { grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 8px; } .player-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 8px; max-height: 46vh; overflow: auto; padding-right: 2px; } .stat-box, .box, .role-mini, .player-card, .log-list div { padding: 10px; border-radius: 16px; } .stat-box strong { font-size: 20px; margin: 4px 0; } input, select { padding: 9px 10px; border-radius: 12px; } .tool-row { margin: 8px 0; } .tool-row input[type="range"] { width: 140px; } .chat-box { height: 120px; padding: 8px; border-radius: 14px; } .log-list { max-height: 125px; overflow: auto; } .side-column .card:nth-child(2) { display: none; } .checkbox-line { padding: 10px; margin: 8px 0; border-radius: 14px; } .role-box { padding: 8px; } .role-box p { display: none; } }
  </style>
</head>
<body>
  <main class="page">
    <div class="container">
      <header class="hero">
        <div>
          <div class="badge">🌙 Ma Sói P2P Online</div>
          <h1>Ma Sói chơi chung bằng mã phòng</h1>
          <p>Một máy làm chủ phòng, máy khác nhập mã để kết nối P2P. Bot tự chơi độc lập theo timer.</p>
          <div class="version">Phiên bản v1.3.1</div>
        </div>
        <div class="hero-actions">
          <button class="btn secondary" id="resetBtn">🔄 Phòng mới</button>
          <button class="btn" id="copyBtn">📋 Copy mã</button>
        </div>
      </header>
      <div class="layout">
        <div class="main-column">
          <section class="card">
            <div class="grid-3">
              <div class="stat-box"><span>📡 Mã phòng</span><strong id="roomCodeText">----</strong><small id="networkHint">Bấm “Tạo phòng” để lấy mã thật.</small></div>
              <div class="stat-box"><span>👥 Người chơi</span><strong id="playerCountText">0/8</strong><small id="botCountText">Bot: 0</small></div>
              <div class="stat-box"><span>☀️ Trạng thái</span><strong id="phaseText">Sảnh</strong><small id="networkStatusText">Offline</small><small id="statusText">Game tiếp tục cho đến khi Làng hoặc Sói thắng.</small></div>
            </div>
          </section>
          <section class="card">
            <div class="section-header"><div><h2>🌐 Phòng online</h2><p>Tạo phòng hoặc nhập mã để vào phòng người khác.</p></div></div>
            <div class="connect-grid">
              <div class="box"><h3>Chủ phòng</h3><input id="hostNameInput" value="Chủ phòng" placeholder="Tên chủ phòng" /><div class="tool-row"><button class="btn success" id="createOnlineRoomBtn">🏠 Tạo phòng</button></div></div>
              <div class="box"><h3>Vào phòng</h3><input id="guestNameInput" placeholder="Tên của bạn" /><input id="joinRoomInput" placeholder="Nhập mã phòng" style="margin-top: 10px;" /><div class="tool-row"><button class="btn blue" id="joinOnlineRoomBtn">🚪 Vào phòng</button></div></div>
            </div>
            <div class="compact-note" id="peerStatusText">P2P dùng PeerJS để hai máy tìm nhau, không dùng Firebase/Supabase.</div>
          </section>
          <section class="card" id="lobbyCard">
            <div class="section-header"><div><h2>➕ Sảnh chờ</h2><p>Chủ phòng thêm người, thêm bot và bắt đầu ván.</p></div><button class="btn success" id="startBtn">▶️ Bắt đầu</button></div>
            <div class="form-grid"><input id="localAddNameInput" placeholder="Tên người chơi thêm thủ công" /><button class="btn secondary" id="addLocalHumanBtn">Thêm người</button><button class="btn ghost" id="addBotBtn">🤖 Thêm bot</button><button class="btn ghost" id="fillBotsBtn">Đủ bot</button></div>
            <div class="tool-row"><label for="minPlayersInput">Số người:</label><input id="minPlayersInput" type="range" min="5" max="12" value="8" /><b id="minPlayersText">8</b></div>
            <h3>✨ Vai trò khi đủ <span id="roleTargetText">8</span> người</h3><div class="role-grid" id="roleGrid"></div>
          </section>
          <section class="card hidden" id="gameCard">
            <div id="winnerBox" class="winner-box hidden"></div>
            <div id="nightPanel" class="hidden"><h2>🌙 Pha đêm</h2><div class="compact-note">Bot tự hành động sau vài giây. Chủ phòng chỉ chốt lượt.</div><div class="action-grid"><div class="box"><b>💀 Sói cắn</b><select id="wolfTargetSelect"></select></div><div class="box"><b>🛡️ Bảo vệ</b><select id="doctorTargetSelect"></select></div><div class="box"><b>👁️ Tiên tri</b><select id="seerTargetSelect"></select></div></div><label class="checkbox-line"><input type="checkbox" id="witchSaveInput" />💗 Phù thủy cứu</label><div class="tool-row"><button class="btn" id="resolveNightBtn">☀️ Kết thúc đêm</button></div></div>
            <div id="dayPanel" class="hidden"><h2>☀️ Pha ngày</h2><div class="grid-2"><div class="box"><b>Thông báo</b><p id="nightVictimText">Mọi người thức dậy và bắt đầu thảo luận.</p><p id="seerResultText" class="seer-result"></p></div><div class="box"><b>Bỏ phiếu</b><p>Người thật tự chọn. Bot tự vote sau vài giây.</p></div></div><div class="vote-list" id="voteList"></div><div class="tool-row"><button class="btn danger" id="resolveVoteBtn">💀 Chốt phiếu</button></div></div>
          </section>
          <section class="card"><h2>👥 Người chơi</h2><div class="player-grid" id="playerGrid"></div></section>
        </div>
        <aside class="side-column">
          <section class="card"><h2>💬 Chat phòng</h2><div class="chat-box" id="chatBox"></div><div class="chat-input"><input id="chatInput" placeholder="Nhập tin nhắn..." /><button class="btn" id="sendChatBtn">Gửi</button></div></section>
          <section class="card"><h2>👑 Luật thắng</h2><p><b>Làng:</b> loại hết Ma Sói.</p><p><b>Sói:</b> số sói còn sống bằng hoặc nhiều hơn dân làng.</p><p><b>Bot:</b> tự hành động, không nghe điều khiển người chơi.</p></section>
          <section class="card"><h2>📜 Nhật ký</h2><div class="log-list" id="logList"></div></section>
        </aside>
      </div>
    </div>
  </main>
  <script src="https://unpkg.com/peerjs@1.5.5/dist/peerjs.min.js"></script>
  <script>
    "use strict";
    const VERSION = "v1.3.1";
    const PEER_PREFIX = "ma-soi-p2p-";
    const BOT_THINK_MS = 2200;
    const ROLE_DATA = {
      wolf: { name: "Ma sói", team: "Sói", icon: "🐺", desc: "Mỗi đêm cùng bầy sói chọn một người để cắn." },
      villager: { name: "Dân làng", team: "Làng", icon: "🧑‍🌾", desc: "Không có kỹ năng ban đêm, thắng bằng suy luận và bỏ phiếu đúng." },
      seer: { name: "Tiên tri", team: "Làng", icon: "🔮", desc: "Mỗi đêm soi một người để biết họ thuộc phe Sói hay không." },
      doctor: { name: "Bảo vệ", team: "Làng", icon: "🛡️", desc: "Mỗi đêm bảo vệ một người khỏi bị sói cắn." },
      witch: { name: "Phù thủy", team: "Làng", icon: "🧪", desc: "Có một bình cứu. Bản đơn giản cho phép cứu nạn nhân đêm đó." },
      hunter: { name: "Thợ săn", team: "Làng", icon: "🏹", desc: "Vai mạnh trong làng. Bản này hiển thị vai trước, chưa xử lý bắn khi chết." },
      cupid: { name: "Thần tình yêu", team: "Làng", icon: "💘", desc: "Vai đặc biệt. Bản này hiển thị vai trước, chưa xử lý ghép đôi." },
      guard: { name: "Hiệp sĩ", team: "Làng", icon: "⚔️", desc: "Có tiếng nói mạnh khi tranh luận. Bản này tính phiếu như người thường." },
      alphaWolf: { name: "Sói đầu đàn", team: "Sói", icon: "👑🐺", desc: "Sói đặc biệt, cùng phe Sói đi cắn mỗi đêm." }
    };
    const BOT_NAMES = ["An Bot", "Bình Bot", "Chi Bot", "Dũng Bot", "Em Bot", "Giang Bot", "Hạ Bot", "Khoa Bot", "Linh Bot", "Minh Bot", "Nhi Bot", "Phong Bot", "Quân Bot", "Trang Bot", "Vy Bot", "Long Bot", "Tú Bot", "Huy Bot"];
    const ROLE_SET_BY_SIZE = {
      5: ["wolf", "seer", "doctor", "villager", "villager"], 6: ["wolf", "seer", "doctor", "hunter", "villager", "villager"], 7: ["wolf", "wolf", "seer", "doctor", "hunter", "villager", "villager"], 8: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "villager", "villager"], 9: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"], 10: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"], 11: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager"], 12: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager", "villager"]
    };
    const state = { roomCode: makeCode(), localName: "Chủ phòng", localPlayerId: "", minPlayers: 8, players: [], phase: "lobby", day: 0, selectedWolfTarget: "", selectedDoctorTarget: "", selectedSeerTarget: "", selectedWitchSave: false, seerResult: "", nightVictim: "", votes: {}, log: ["Game đã mở. Tạo phòng online hoặc vào phòng bằng mã."], chat: [{ from: "Hệ thống", text: "Chào mừng đến với Ma Sói P2P!" }], botTimer: null, network: { mode: "offline", status: "Offline", peerReady: false, hostPeerId: "", connections: {} } };
    let peer = null;
    let hostConnection = null;
    const els = {};
    document.addEventListener("DOMContentLoaded", init);
    function init() { cacheElements(); const ok = runSelfTests(); if (!ok) return; bindEvents(); setupOfflineHost(); render(); }
    function cacheElements() { ["resetBtn","copyBtn","roomCodeText","networkHint","playerCountText","networkStatusText","statusText","peerStatusText","botCountText","hostNameInput","guestNameInput","joinRoomInput","createOnlineRoomBtn","joinOnlineRoomBtn","phaseText","lobbyCard","gameCard","winnerBox","nightPanel","dayPanel","localAddNameInput","addLocalHumanBtn","addBotBtn","fillBotsBtn","startBtn","minPlayersInput","minPlayersText","roleTargetText","roleGrid","wolfTargetSelect","doctorTargetSelect","seerTargetSelect","witchSaveInput","resolveNightBtn","nightVictimText","seerResultText","voteList","resolveVoteBtn","playerGrid","chatBox","chatInput","sendChatBtn","logList"].forEach((id) => { els[id] = document.getElementById(id); }); }
    function mustEl(name) { return els[name] || null; }
    function setText(name, value) { const el = mustEl(name); if (el) el.textContent = value; }
    function setHtml(name, value) { const el = mustEl(name); if (el) el.innerHTML = value; }
    function bindEvents() { on("resetBtn","click",resetGame); on("copyBtn","click",copyInvite); on("createOnlineRoomBtn","click",createOnlineRoom); on("joinOnlineRoomBtn","click",joinOnlineRoom); on("addLocalHumanBtn","click",addLocalHuman); on("addBotBtn","click",() => addBot(1)); on("fillBotsBtn","click",fillBots); on("startBtn","click",startGame); on("resolveNightBtn","click",resolveNight); on("resolveVoteBtn","click",resolveVote); on("sendChatBtn","click",sendChat); on("chatInput","keydown",(event) => { if (event.key === "Enter") sendChat(); }); on("minPlayersInput","input",() => { if (!isHost()) return; state.minPlayers = Number(els.minPlayersInput.value); syncAfterHostChange(); }); on("witchSaveInput","change",() => { if (!isHost()) return; state.selectedWitchSave = els.witchSaveInput.checked; syncAfterHostChange(); }); on("wolfTargetSelect","change",() => { if (!isHost()) return; state.selectedWolfTarget = els.wolfTargetSelect.value; syncAfterHostChange(); }); on("doctorTargetSelect","change",() => { if (!isHost()) return; state.selectedDoctorTarget = els.doctorTargetSelect.value; syncAfterHostChange(); }); on("seerTargetSelect","change",() => { if (!isHost()) return; state.selectedSeerTarget = els.seerTargetSelect.value; syncAfterHostChange(); }); }
    function on(id, eventName, handler) { const el = els[id]; if (el) el.addEventListener(eventName, handler); }
    function setupOfflineHost() { const host = createPlayer("Chủ phòng", false, true, "local"); state.localPlayerId = host.id; state.players = [host]; }
    function makeCode() { return Math.random().toString(36).slice(2, 6).toUpperCase() + "-" + Math.random().toString(36).slice(2, 6).toUpperCase(); }
    function makePeerIdFromCode(code) { return PEER_PREFIX + String(code || "").toLowerCase().replace(/[^a-z0-9-]/g, ""); }
    function makeId() { return window.crypto && typeof window.crypto.randomUUID === "function" ? window.crypto.randomUUID() : "id-" + Date.now() + "-" + Math.random().toString(36).slice(2); }
    function createPlayer(name, isBot, host, connectionId) { return { id: makeId(), name, isBot: Boolean(isBot), host: Boolean(host), connectionId: connectionId || "", alive: true, role: null }; }
    function shuffle(arr) { return arr.slice().sort(() => Math.random() - 0.5); }
    function chooseRandom(list) { return Array.isArray(list) && list.length ? list[Math.floor(Math.random() * list.length)] : null; }
    function getRoleSet(count) { const safeCount = Math.max(5, Math.min(12, Number(count) || 5)); return ROLE_SET_BY_SIZE[safeCount] || ROLE_SET_BY_SIZE[12]; }
    function isWolfRole(role) { return role === "wolf" || role === "alphaWolf"; }
    function alivePlayers() { return state.players.filter((player) => player.alive); }
    function wolvesAlive() { return state.players.filter((player) => player.alive && isWolfRole(player.role)); }
    function villagersAlive() { return state.players.filter((player) => player.alive && !isWolfRole(player.role)); }
    function isHost() { return state.network.mode === "host" || state.network.mode === "offline"; }
    function getWinner() { if (state.phase === "lobby") return ""; if (wolvesAlive().length === 0) return "Làng thắng! Tất cả ma sói đã bị loại."; if (wolvesAlive().length >= villagersAlive().length) return "Sói thắng! Số sói đã bằng hoặc vượt số dân làng."; return ""; }
    function syncAfterHostChange() { scheduleBotBrain(); broadcastState(); render(); }
    function pushLog(text, shouldBroadcast) { state.log.unshift(text); state.log = state.log.slice(0, 16); if (shouldBroadcast !== false) broadcastState(); render(); }
    function resetGame() { clearBotTimer(); closeNetwork(); state.roomCode = makeCode(); state.localName = "Chủ phòng"; state.minPlayers = 8; state.phase = "lobby"; state.day = 0; state.selectedWolfTarget = ""; state.selectedDoctorTarget = ""; state.selectedSeerTarget = ""; state.selectedWitchSave = false; state.seerResult = ""; state.nightVictim = ""; state.votes = {}; state.log = ["Phòng mới đã được tạo. Tạo phòng online hoặc chơi offline."]; state.chat = [{ from: "Hệ thống", text: "Phòng mới đã sẵn sàng." }]; setupOfflineHost(); render(); }
    function closeNetwork() { Object.values(state.network.connections).forEach((conn) => { try { conn.close(); } catch (error) {} }); state.network.connections = {}; if (hostConnection) { try { hostConnection.close(); } catch (error) {} hostConnection = null; } if (peer) { try { peer.destroy(); } catch (error) {} peer = null; } state.network.mode = "offline"; state.network.status = "Offline"; state.network.peerReady = false; state.network.hostPeerId = ""; }
    async function copyInvite() { const invite = "Mời bạn vào chơi Ma Sói P2P! Mã phòng: " + state.roomCode + ". Mở link game, nhập tên và nhập mã này để vào phòng."; try { if (navigator.clipboard && navigator.clipboard.writeText) { await navigator.clipboard.writeText(invite); setText("copyBtn", "✅ Đã copy"); setTimeout(() => render(), 1200); } else pushLog("Trình duyệt không hỗ trợ copy tự động. Mã phòng là " + state.roomCode + ".", false); } catch (error) { pushLog("Không copy tự động được. Mã phòng là " + state.roomCode + ".", false); } }
    function requirePeerLibrary() { if (typeof Peer === "undefined") { pushLog("Không tải được PeerJS. Hãy kiểm tra internet/adblock rồi tải lại trang.", false); return false; } return true; }
    function createOnlineRoom() { if (!requirePeerLibrary()) return; clearBotTimer(); closeNetwork(); const hostName = cleanName(els.hostNameInput.value || "Chủ phòng"); state.roomCode = makeCode(); const hostPlayer = createPlayer(hostName, false, true, "host"); state.localPlayerId = hostPlayer.id; state.localName = hostName; state.players = [hostPlayer]; state.phase = "lobby"; state.day = 0; state.votes = {}; state.network.mode = "host"; state.network.status = "Đang tạo phòng"; state.network.hostPeerId = makePeerIdFromCode(state.roomCode); render(); peer = new Peer(state.network.hostPeerId, { debug: 1 }); peer.on("open", () => { state.network.peerReady = true; state.network.status = "Đang mở phòng"; pushLog("Phòng online đã mở. Mã phòng: " + state.roomCode + ".", false); render(); }); peer.on("connection", setupHostConnection); peer.on("error", (error) => { state.network.status = "Lỗi"; pushLog("Lỗi tạo phòng online: " + readablePeerError(error), false); render(); }); }
    function joinOnlineRoom() { if (!requirePeerLibrary()) return; const guestName = cleanName(els.guestNameInput.value); const code = els.joinRoomInput.value.trim().toUpperCase(); if (!guestName) { pushLog("Hãy nhập tên của bạn trước khi vào phòng.", false); return; } if (!code) { pushLog("Hãy nhập mã phòng trước khi vào.", false); return; } clearBotTimer(); closeNetwork(); state.roomCode = code; state.localName = guestName; state.network.mode = "guest"; state.network.status = "Đang vào phòng"; state.network.hostPeerId = makePeerIdFromCode(code); state.players = []; render(); peer = new Peer(undefined, { debug: 1 }); peer.on("open", () => { state.network.peerReady = true; hostConnection = peer.connect(state.network.hostPeerId, { reliable: true }); setupGuestConnection(hostConnection, guestName); }); peer.on("error", (error) => { state.network.status = "Lỗi"; pushLog("Lỗi vào phòng: " + readablePeerError(error), false); render(); }); }
    function setupHostConnection(conn) { state.network.connections[conn.peer] = conn; conn.on("open", () => { sendTo(conn, { type: "hello-host" }); broadcastState(); render(); }); conn.on("data", (data) => handleHostData(conn, safeMessage(data))); conn.on("close", () => { delete state.network.connections[conn.peer]; const player = state.players.find((item) => item.connectionId === conn.peer); if (player) pushLog(player.name + " đã mất kết nối.", false); broadcastState(); render(); }); conn.on("error", () => { delete state.network.connections[conn.peer]; pushLog("Một kết nối bị lỗi.", false); render(); }); }
    function setupGuestConnection(conn, guestName) { conn.on("open", () => { state.network.status = "Đã kết nối"; sendTo(conn, { type: "join", name: guestName }); pushLog("Đã gửi yêu cầu vào phòng.", false); render(); }); conn.on("data", (data) => handleGuestData(safeMessage(data))); conn.on("close", () => { state.network.status = "Mất kết nối"; pushLog("Đã mất kết nối với chủ phòng.", false); render(); }); conn.on("error", () => { state.network.status = "Lỗi kết nối"; pushLog("Kết nối tới chủ phòng bị lỗi.", false); render(); }); }
    function handleHostData(conn, message) { if (!message || !message.type) return; if (message.type === "join") { if (state.players.length >= 12) { sendTo(conn, { type: "reject", reason: "Phòng đã đủ 12 người." }); return; } let player = state.players.find((item) => item.connectionId === conn.peer); if (!player) { player = createPlayer(cleanName(message.name || "Người chơi"), false, false, conn.peer); state.players.push(player); pushLog(player.name + " đã vào phòng online.", false); } sendTo(conn, { type: "joined", state: publicState(), playerId: player.id }); broadcastState(); render(); return; } if (message.type === "chat") { addChat(cleanName(message.from || "Người chơi"), String(message.text || ""), false); broadcastState(); render(); return; } if (message.type === "vote" && state.phase === "day") { state.votes[message.voterId] = message.targetId; broadcastState(); render(); } }
    function handleGuestData(message) { if (!message || !message.type) return; if (message.type === "state" || message.type === "joined") { if (message.playerId) state.localPlayerId = message.playerId; applyPublicState(message.state); state.network.status = "Đã kết nối"; render(); return; } if (message.type === "reject") { state.network.status = "Bị từ chối"; pushLog(message.reason || "Không thể vào phòng.", false); render(); } }
    function publicState() { return { roomCode: state.roomCode, minPlayers: state.minPlayers, players: state.players, phase: state.phase, day: state.day, selectedWolfTarget: state.selectedWolfTarget, selectedDoctorTarget: state.selectedDoctorTarget, selectedSeerTarget: state.selectedSeerTarget, selectedWitchSave: state.selectedWitchSave, seerResult: state.seerResult, nightVictim: state.nightVictim, votes: state.votes, log: state.log, chat: state.chat }; }
    function applyPublicState(next) { if (!next) return; state.roomCode = next.roomCode || state.roomCode; state.minPlayers = Number(next.minPlayers) || state.minPlayers; state.players = Array.isArray(next.players) ? next.players : state.players; state.phase = next.phase || state.phase; state.day = Number(next.day) || 0; state.selectedWolfTarget = next.selectedWolfTarget || ""; state.selectedDoctorTarget = next.selectedDoctorTarget || ""; state.selectedSeerTarget = next.selectedSeerTarget || ""; state.selectedWitchSave = Boolean(next.selectedWitchSave); state.seerResult = next.seerResult || ""; state.nightVictim = next.nightVictim || ""; state.votes = next.votes || {}; state.log = Array.isArray(next.log) ? next.log : state.log; state.chat = Array.isArray(next.chat) ? next.chat : state.chat; }
    function broadcastState() { if (state.network.mode !== "host") return; const msg = { type: "state", state: publicState() }; Object.values(state.network.connections).forEach((conn) => sendTo(conn, msg)); }
    function sendTo(conn, message) { try { if (conn && conn.open) conn.send(message); } catch (error) { pushLog("Gửi dữ liệu thất bại.", false); } }
    function safeMessage(data) { return data && typeof data === "object" ? data : null; }
    function readablePeerError(error) { const type = error && error.type ? error.type : "unknown"; if (type === "unavailable-id") return "Mã phòng này đang được dùng. Hãy bấm tạo phòng mới."; if (type === "peer-unavailable") return "Không tìm thấy phòng. Kiểm tra mã hoặc chủ phòng đã tắt."; if (type === "network") return "Lỗi mạng hoặc server tín hiệu không phản hồi."; return type; }
    function cleanName(value) { return String(value || "").trim().slice(0, 24) || "Người chơi"; }
    function addLocalHuman() { if (!isHost()) return; if (state.players.length >= 12) { pushLog("Phòng đã đủ 12 người."); return; } const name = cleanName(els.localAddNameInput.value); state.players.push(createPlayer(name, false, false, "manual")); els.localAddNameInput.value = ""; pushLog(name + " đã được thêm thủ công."); scheduleBotBrain(); }
    function addBot(count) { if (!isHost()) return; if (state.players.length >= 12) { pushLog("Phòng đã đủ 12 người, không thể thêm bot."); return; } const amount = Math.min(Number(count) || 1, 12 - state.players.length); const used = new Set(state.players.map((player) => player.name)); for (let i = 0; i < amount; i++) { const botName = BOT_NAMES.find((name) => !used.has(name)) || "Bot " + (state.players.length + 1); used.add(botName); state.players.push(createPlayer(botName, true, false, "bot")); } pushLog("Đã thêm " + amount + " bot. Bot sẽ tự chơi độc lập."); scheduleBotBrain(); }
    function fillBots() { if (!isHost()) return; const need = Math.max(0, state.minPlayers - state.players.length); if (need > 0) addBot(need); else pushLog("Số người đã đủ, không cần thêm bot."); }
    function startGame() { if (!isHost()) return; let roster = state.players.slice(); const need = Math.max(0, state.minPlayers - roster.length); for (let i = 0; i < need; i++) { const botName = BOT_NAMES.find((name) => !roster.some((player) => player.name === name)) || "Bot " + (roster.length + 1); roster.push(createPlayer(botName, true, false, "bot")); } roster = roster.slice(0, 12); const roles = shuffle(getRoleSet(roster.length)); state.players = shuffle(roster).map((player, index) => ({ ...player, role: roles[index], alive: true })); state.phase = "night"; state.day = 1; state.votes = {}; state.nightVictim = ""; state.seerResult = ""; state.selectedWolfTarget = ""; state.selectedDoctorTarget = ""; state.selectedSeerTarget = ""; state.selectedWitchSave = false; pushLog("Game bắt đầu với " + state.players.length + " người. Bot sẽ tự hành động trong đêm."); scheduleBotBrain(); }
    function clearBotTimer() { if (state.botTimer) { clearTimeout(state.botTimer); state.botTimer = null; } }
    function scheduleBotBrain() { clearBotTimer(); if (!isHost() || state.phase === "lobby" || getWinner()) return; state.botTimer = setTimeout(() => { state.botTimer = null; runBotBrain(); }, BOT_THINK_MS); }
    function runBotBrain() { if (!isHost() || state.phase === "lobby" || getWinner()) return; let changed = false; if (state.phase === "night") changed = botNightActions(); if (state.phase === "day") changed = botDayVotes(); if (changed) { state.log.unshift("Bot đã tự hành động độc lập."); state.log = state.log.slice(0, 16); broadcastState(); render(); } }
    function botNightActions() { const alive = alivePlayers(); const botWolves = alive.filter((p) => p.isBot && isWolfRole(p.role)); const humanWolves = alive.filter((p) => !p.isBot && isWolfRole(p.role)); const botDoctors = alive.filter((p) => p.isBot && p.role === "doctor"); const botSeers = alive.filter((p) => p.isBot && p.role === "seer"); const botWitches = alive.filter((p) => p.isBot && p.role === "witch"); let changed = false; if (!state.selectedWolfTarget && botWolves.length > 0 && humanWolves.length === 0) { const victim = chooseRandom(alive.filter((p) => !isWolfRole(p.role))); if (victim) { state.selectedWolfTarget = victim.id; changed = true; } } if (!state.selectedDoctorTarget && botDoctors.length > 0) { const protect = chooseRandom(alive.filter((p) => !isWolfRole(p.role))) || chooseRandom(alive); if (protect) { state.selectedDoctorTarget = protect.id; changed = true; } } if (!state.selectedSeerTarget && botSeers.length > 0) { const target = chooseRandom(alive.filter((p) => p.role !== "seer")); if (target) { state.selectedSeerTarget = target.id; changed = true; } } if (!state.selectedWitchSave && botWitches.length > 0 && state.selectedWolfTarget && Math.random() < 0.45) { state.selectedWitchSave = true; changed = true; } return changed; }
    function botDayVotes() { const alive = alivePlayers(); const bots = alive.filter((p) => p.isBot); let changed = false; bots.forEach((bot) => { if (state.votes[bot.id]) return; const candidates = alive.filter((p) => p.id !== bot.id); let pool = candidates; if (!isWolfRole(bot.role)) { const wolves = candidates.filter((p) => isWolfRole(p.role)); if (wolves.length && Math.random() < 0.55) pool = wolves; } else { const nonWolves = candidates.filter((p) => !isWolfRole(p.role)); if (nonWolves.length) pool = nonWolves; } const pick = chooseRandom(pool); if (pick) { state.votes[bot.id] = pick.id; changed = true; } }); return changed; }
    function resolveNight() { if (!isHost()) return; botNightActions(); const wolfTarget = state.players.find((player) => player.id === state.selectedWolfTarget); const doctorTarget = state.players.find((player) => player.id === state.selectedDoctorTarget); const seerTarget = state.players.find((player) => player.id === state.selectedSeerTarget); let victim = null; if (wolfTarget) { const savedByDoctor = doctorTarget && doctorTarget.id === wolfTarget.id; const savedByWitch = state.selectedWitchSave; if (!savedByDoctor && !savedByWitch) { victim = wolfTarget; state.players = state.players.map((player) => player.id === wolfTarget.id ? { ...player, alive: false } : player); } } state.seerResult = seerTarget ? seerTarget.name + " " + (isWolfRole(seerTarget.role) ? "thuộc phe Sói" : "không thuộc phe Sói") + "." : ""; state.nightVictim = victim ? victim.name + " đã bị sói cắn." : "Không ai chết trong đêm nay."; state.phase = "day"; state.votes = {}; pushLog(victim ? "Sáng ngày " + state.day + ": " + victim.name + " đã chết." : "Sáng ngày " + state.day + ": không ai chết."); scheduleBotBrain(); }
    function resolveVote() { if (!isHost()) return; botDayVotes(); const count = {}; Object.values(state.votes).forEach((targetId) => { if (targetId) count[targetId] = (count[targetId] || 0) + 1; }); const sorted = Object.entries(count).sort((a, b) => b[1] - a[1]); if (!sorted.length) { state.phase = "night"; state.day += 1; pushLog("Không đủ phiếu, không ai bị treo cổ. Đêm tiếp theo bắt đầu."); scheduleBotBrain(); return; } const targetId = sorted[0][0]; const amount = sorted[0][1]; const eliminated = state.players.find((player) => player.id === targetId); state.players = state.players.map((player) => player.id === targetId ? { ...player, alive: false } : player); state.phase = "night"; state.day += 1; state.selectedWolfTarget = ""; state.selectedDoctorTarget = ""; state.selectedSeerTarget = ""; state.selectedWitchSave = false; state.nightVictim = ""; state.votes = {}; pushLog((eliminated ? eliminated.name : "Một người") + " bị treo cổ với " + amount + " phiếu. Đêm tiếp theo bắt đầu."); scheduleBotBrain(); }
    function sendChat() { const text = els.chatInput.value.trim(); if (!text) return; const from = state.localName || "Người chơi"; if (state.network.mode === "guest" && hostConnection && hostConnection.open) sendTo(hostConnection, { type: "chat", from, text }); else addChat(from, text, true); els.chatInput.value = ""; render(); }
    function addChat(from, text, shouldBroadcast) { state.chat.push({ from: cleanName(from), text: String(text).slice(0, 200) }); state.chat = state.chat.slice(-10); if (shouldBroadcast !== false) broadcastState(); }
    function castVote(voterId, targetId) { if (state.network.mode === "guest") sendTo(hostConnection, { type: "vote", voterId, targetId }); else if (isHost()) { state.votes[voterId] = targetId; syncAfterHostChange(); } }
    function render() { const winner = getWinner(); const bots = state.players.filter((player) => player.isBot).length; setText("roomCodeText", state.roomCode); setText("playerCountText", state.players.length + "/" + state.minPlayers); setText("networkStatusText", state.network.status); setText("peerStatusText", state.network.mode === "host" ? "Bạn là chủ phòng. Gửi mã cho bạn bè." : state.network.mode === "guest" ? "Bạn đang nhận dữ liệu từ chủ phòng." : "P2P dùng PeerJS để hai máy tìm nhau, không dùng Firebase/Supabase."); setText("networkHint", state.network.mode === "host" ? "Mã này cho người khác nhập để vào phòng." : state.network.mode === "guest" ? "Bạn đang ở phòng " + state.roomCode + "." : "Bấm “Tạo phòng” để lấy mã thật."); setText("botCountText", "Bot: " + bots); setText("phaseText", state.phase === "lobby" ? "Sảnh" : state.phase === "night" ? "Đêm " + state.day : "Ngày " + state.day); setText("statusText", winner || "Game tiếp tục cho đến khi Làng hoặc Sói thắng."); if (els.minPlayersInput) els.minPlayersInput.value = String(state.minPlayers); setText("minPlayersText", String(state.minPlayers)); setText("roleTargetText", String(state.minPlayers)); if (els.witchSaveInput) els.witchSaveInput.checked = state.selectedWitchSave; setControlsEnabled(); toggle(els.lobbyCard, state.phase !== "lobby"); toggle(els.gameCard, state.phase === "lobby"); toggle(els.winnerBox, !winner); toggle(els.nightPanel, state.phase !== "night" || Boolean(winner)); toggle(els.dayPanel, state.phase !== "day" || Boolean(winner)); if (winner && els.winnerBox) els.winnerBox.innerHTML = "<h2>" + escapeHtml(winner) + "</h2><p>Bấm “Phòng mới” để chơi ván khác.</p>"; renderRoles(); renderNightSelects(); renderVotes(); renderPlayers(); renderChat(); renderLog(); setText("nightVictimText", state.nightVictim || "Mọi người thức dậy và bắt đầu thảo luận."); setText("seerResultText", state.seerResult ? "Kết quả riêng của Tiên tri: " + state.seerResult : ""); }
    function setControlsEnabled() { [els.addLocalHumanBtn, els.addBotBtn, els.fillBotsBtn, els.startBtn, els.minPlayersInput, els.wolfTargetSelect, els.doctorTargetSelect, els.seerTargetSelect, els.witchSaveInput, els.resolveNightBtn, els.resolveVoteBtn].forEach((el) => { if (el) el.disabled = !isHost(); }); if (els.createOnlineRoomBtn) els.createOnlineRoomBtn.disabled = state.network.mode === "host" && state.network.peerReady; if (els.joinOnlineRoomBtn) els.joinOnlineRoomBtn.disabled = state.network.mode === "guest" && state.network.status === "Đã kết nối"; }
    function renderRoles() { const counts = {}; getRoleSet(state.minPlayers).forEach((role) => { counts[role] = (counts[role] || 0) + 1; }); setHtml("roleGrid", Object.keys(counts).map((role) => { const data = ROLE_DATA[role]; return "<div class=\"role-mini\"><span>" + data.icon + "</span><b>" + escapeHtml(data.name) + " x" + counts[role] + "</b><small>Phe " + escapeHtml(data.team) + "</small></div>"; }).join("")); }
    function renderNightSelects() { const alive = alivePlayers(); fillSelect(els.wolfTargetSelect, alive.filter((player) => !isWolfRole(player.role)), state.selectedWolfTarget, "Chưa chọn"); fillSelect(els.doctorTargetSelect, alive, state.selectedDoctorTarget, "Chưa chọn"); fillSelect(els.seerTargetSelect, alive, state.selectedSeerTarget, "Chưa chọn"); }
    function fillSelect(selectEl, players, selectedValue, emptyLabel) { if (!selectEl) return; selectEl.innerHTML = "<option value=\"\">" + escapeHtml(emptyLabel) + "</option>"; players.forEach((player) => { const option = document.createElement("option"); option.value = player.id; option.textContent = player.name; selectEl.appendChild(option); }); selectEl.value = selectedValue; }
    function renderVotes() { const alive = alivePlayers(); if (!els.voteList) return; els.voteList.innerHTML = ""; alive.forEach((voter) => { const row = document.createElement("div"); row.className = "vote-row"; const label = document.createElement("b"); label.textContent = voter.name + (voter.isBot ? " BOT" : ""); const select = document.createElement("select"); select.innerHTML = "<option value=\"\">Chọn người bị nghi</option>"; alive.filter((player) => player.id !== voter.id).forEach((candidate) => { const option = document.createElement("option"); option.value = candidate.id; option.textContent = candidate.name; select.appendChild(option); }); select.value = state.votes[voter.id] || ""; select.disabled = voter.isBot || (!isHost() && voter.id !== state.localPlayerId); select.addEventListener("change", () => castVote(voter.id, select.value)); row.appendChild(label); row.appendChild(select); els.voteList.appendChild(row); }); }
    function renderPlayers() { setHtml("playerGrid", state.players.map((player) => { const role = player.role ? ROLE_DATA[player.role] : null; const status = (player.host ? "Chủ phòng " : "") + (player.isBot ? "BOT tự chơi " : "") + (player.id === state.localPlayerId ? "Bạn " : "") + (player.alive ? "Còn sống" : "Đã chết"); const roleBox = role ? "<div class=\"role-box\"><b>" + escapeHtml(role.name) + "</b><small>Phe " + escapeHtml(role.team) + "</small><p>" + escapeHtml(role.desc) + "</p></div>" : ""; return "<div class=\"player-card " + (player.alive ? "" : "dead") + "\"><div class=\"player-top\"><div><b>" + escapeHtml(player.name) + "</b><small>" + escapeHtml(status) + "</small></div><span>" + (role ? role.icon : player.isBot ? "🤖" : "🙂") + "</span></div>" + roleBox + "</div>"; }).join("")); }
    function renderChat() { if (!els.chatBox) return; els.chatBox.innerHTML = state.chat.map((m) => "<p><b>" + escapeHtml(m.from) + ":</b> " + escapeHtml(m.text) + "</p>").join(""); els.chatBox.scrollTop = els.chatBox.scrollHeight; }
    function renderLog() { setHtml("logList", state.log.map((entry) => "<div>" + escapeHtml(entry) + "</div>").join("")); }
    function toggle(el, shouldHide) { if (el) el.classList.toggle("hidden", shouldHide); }
    function escapeHtml(value) { return String(value).replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/\"/g, "&quot;").replace(/'/g, "&#039;"); }
    function runSelfTests() { const required = ["resetBtn","copyBtn","roomCodeText","networkHint","playerCountText","networkStatusText","statusText","peerStatusText","botCountText","hostNameInput","guestNameInput","joinRoomInput","createOnlineRoomBtn","joinOnlineRoomBtn","phaseText","lobbyCard","gameCard","winnerBox","nightPanel","dayPanel","localAddNameInput","addLocalHumanBtn","addBotBtn","fillBotsBtn","startBtn","minPlayersInput","minPlayersText","roleTargetText","roleGrid","wolfTargetSelect","doctorTargetSelect","seerTargetSelect","witchSaveInput","resolveNightBtn","nightVictimText","seerResultText","voteList","resolveVoteBtn","playerGrid","chatBox","chatInput","sendChatBtn","logList"]; const missing = required.filter((id) => !els[id]); const bot = createPlayer("Bot Test", true, false, "bot"); const tests = [ { name: "Không thiếu DOM ID", pass: missing.length === 0 }, { name: "Phiên bản đúng v1.3.1", pass: VERSION === "v1.3.1" }, { name: "Bộ 5 người có đúng 5 vai", pass: getRoleSet(5).length === 5 }, { name: "Bộ 8 người có Sói đầu đàn", pass: getRoleSet(8).includes("alphaWolf") }, { name: "Số quá lớn được giới hạn về 12 vai", pass: getRoleSet(99).length === 12 }, { name: "Chọn ngẫu nhiên danh sách rỗng trả về null", pass: chooseRandom([]) === null }, { name: "Bot được đánh dấu là người máy", pass: bot.isBot === true }, { name: "Bot có connectionId riêng là bot", pass: bot.connectionId === "bot" }, { name: "Hàm kiểm tra PeerJS tồn tại", pass: typeof requirePeerLibrary === "function" } ]; tests.forEach((test) => console.assert(test.pass, "Self-test failed: " + test.name)); if (missing.length) { document.body.innerHTML = "<pre style='color:#fca5a5;background:#020617;padding:20px;white-space:pre-wrap'>Lỗi DOM: thiếu ID sau:\n" + missing.join("\n") + "</pre>"; return false; } if (tests.every((test) => test.pass)) console.log("Ma Sói P2P " + VERSION + ": self-test passed"); return tests.every((test) => test.pass); }
  </script>
</body>
</html>
    .page {
      min-height: 100vh;
      padding: 24px;
      background:
        radial-gradient(circle at top left, rgba(147, 51, 234, 0.24), transparent 34%),
        radial-gradient(circle at top right, rgba(14, 165, 233, 0.16), transparent 30%),
        var(--bg);
    }
    .container { max-width: 1280px; margin: 0 auto; }
    .hero {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      gap: 22px;
      margin-bottom: 22px;
    }
    .badge {
      display: inline-flex;
      gap: 8px;
      align-items: center;
      color: #e9d5ff;
      background: rgba(168, 85, 247, 0.14);
      border: 1px solid rgba(216, 180, 254, 0.22);
      border-radius: 999px;
      padding: 8px 12px;
      font-size: 14px;
    }
    h1 { font-size: clamp(32px, 5vw, 56px); line-height: 1; margin: 12px 0; }
    h2 { margin: 0 0 16px; }
    h3 { margin: 18px 0 10px; }
    p { color: var(--soft); }
    .hero p { max-width: 700px; margin: 0; }
    .hero-actions, .tool-row, .chat-input, .inline-actions {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      align-items: center;
    }
    .layout { display: grid; grid-template-columns: 1fr 360px; gap: 20px; }
    .main-column, .side-column { display: flex; flex-direction: column; gap: 20px; }
    .card {
      background: var(--card);
      border: 1px solid var(--line);
      border-radius: 28px;
      padding: 22px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.24);
    }
    .btn {
      min-height: 44px;
      border-radius: 18px;
      padding: 11px 16px;
      color: white;
      background: var(--purple);
      cursor: pointer;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
      transition: transform 0.15s, opacity 0.15s, background 0.15s;
      user-select: none;
    }
    .btn:hover { transform: translateY(-1px); background: #8b5cf6; }
    .btn.secondary { background: #334155; }
    .btn.success { background: var(--green); }
    .btn.danger { background: var(--red); }
    .btn.blue { background: var(--blue); }
    .btn.ghost { background: transparent; border: 1px solid #334155; }
    .btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }
    .stats-grid, .info-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }
    .info-grid { grid-template-columns: repeat(2, 1fr); margin: 16px 0; }
    .stat-box, .info-box, .action-box, .role-mini, .player-card, .log-list div, .test-row {
      background: var(--card-2);
      border: 1px solid var(--line);
      border-radius: 22px;
      padding: 16px;
    }
    .stat-box span { display: block; color: var(--muted); font-size: 14px; }
    .stat-box strong { display: block; font-size: 32px; margin: 8px 0; color: #d8b4fe; }
    small { display: block; color: var(--muted); }
    input, select, textarea {
      width: 100%;
      background: var(--card-2);
      border: 1px solid #334155;
      color: var(--text);
      border-radius: 16px;
      padding: 12px 14px;
      outline: none;
    }
    input:focus, select:focus, textarea:focus { border-color: var(--purple-2); }
    .section-header { display: flex; justify-content: space-between; gap: 16px; align-items: flex-start; margin-bottom: 18px; }
    .section-header p { margin: 4px 0 0; color: var(--muted); }
    .form-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
    .tool-row { margin: 16px 0; }
    .tool-row input[type="range"] { width: 220px; padding: 0; }
    .role-grid, .action-grid, .player-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
    .action-grid { grid-template-columns: repeat(3, 1fr); margin: 16px 0; }
    .player-grid { grid-template-columns: repeat(3, 1fr); }
    .role-mini span { font-size: 26px; }
    .role-mini b { display: block; margin-top: 6px; }
    .action-box b { display: block; margin-bottom: 10px; }
    .checkbox-line { display: flex; align-items: center; gap: 10px; background: var(--card-2); border: 1px solid var(--line); border-radius: 20px; padding: 14px; margin: 14px 0; }
    .checkbox-line input { width: auto; }
    .winner-box { background: rgba(16, 185, 129, 0.14); border: 1px solid rgba(110, 231, 183, 0.3); border-radius: 24px; padding: 22px; }
    .winner-box h2 { color: #a7f3d0; font-size: 28px; }
    .seer-result { color: #d8b4fe; }
    .vote-list { display: flex; flex-direction: column; gap: 10px; }
    .vote-row { display: grid; grid-template-columns: 1fr 2fr; gap: 10px; align-items: center; background: var(--card-2); border: 1px solid var(--line); border-radius: 18px; padding: 12px; }
    .vote-row small { display: inline; margin-left: 8px; }
    .player-card.dead { background: rgba(127, 29, 29, 0.26); border-color: rgba(248, 113, 113, 0.35); opacity: 0.75; }
    .player-top { display: flex; justify-content: space-between; gap: 12px; }
    .player-top b { font-size: 18px; }
    .player-top span { font-size: 28px; }
    .role-box { margin-top: 12px; background: #0f172a; border-radius: 16px; padding: 12px; }
    .role-box p { color: var(--muted); font-size: 13px; margin-bottom: 0; }
    .chat-box { height: 230px; overflow: auto; background: var(--card-2); border: 1px solid var(--line); border-radius: 18px; padding: 12px; margin-bottom: 10px; }
    .chat-box p { margin: 0 0 8px; color: var(--soft); }
    .chat-box b { color: #d8b4fe; }
    .log-list { display: flex; flex-direction: column; gap: 10px; }
    .log-list div { color: var(--soft); font-size: 14px; }
    .hidden { display: none !important; }
    .notice { border-left: 4px solid var(--amber); padding: 10px 12px; background: rgba(245, 158, 11, 0.1); border-radius: 14px; color: #fde68a; }
    .version { color: var(--muted); margin-top: 8px; font-size: 13px; }
    @media (max-width: 1050px) {
      .layout { grid-template-columns: 1fr; }
      .stats-grid, .role-grid, .player-grid { grid-template-columns: repeat(2, 1fr); }
      .form-grid, .action-grid, .info-grid { grid-template-columns: 1fr; }
    }
    @media (max-width: 640px) {
      .page { padding: 14px; }
      .hero, .section-header { flex-direction: column; }
      .stats-grid, .role-grid, .player-grid { grid-template-columns: 1fr; }
      .vote-row { grid-template-columns: 1fr; }
      .tool-row input[type="range"] { width: 100%; }
      .btn { width: 100%; }
    }
  </style>
</head>
<body>
  <main class="page">
    <div class="container">
      <header class="hero">
        <div>
          <div class="badge">🌙 Ma Sói Web Demo</div>
          <h1>Trò chơi Ma Sói nhiều vai trò</h1>
          <p>Tạo phòng, chia mã mời bạn bè, tự thêm bot khi thiếu người và chơi theo vòng Đêm - Ngày - Bỏ phiếu.</p>
          <div class="version">Phiên bản v1.0.0</div>
        </div>
        <div class="hero-actions">
          <button class="btn secondary" id="resetBtn">🔄 Phòng mới</button>
          <button class="btn" id="copyBtn">📋 Copy lời mời</button>
        </div>
      </header>

      <div class="layout">
        <div class="main-column">
          <section class="card">
            <div class="stats-grid">
              <div class="stat-box">
                <span>📡 Mã phòng</span>
                <strong id="roomCodeText">----</strong>
                <small>Gửi mã này cho bạn bè. Bản HTML/Vercel-only chỉ mô phỏng nhập phòng trên cùng trình duyệt.</small>
              </div>
              <div class="stat-box">
                <span>👥 Người chơi</span>
                <strong id="playerCountText">0/8</strong>
                <small>Tối thiểu 5, tối đa 12. Thiếu người thì game tự thêm bot.</small>
              </div>
              <div class="stat-box">
                <span>☀️ Trạng thái</span>
                <strong id="phaseText">Sảnh</strong>
                <small id="statusText">Game tiếp tục cho đến khi Làng hoặc Sói thắng.</small>
              </div>
            </div>
          </section>

          <section class="card" id="lobbyCard">
            <div class="section-header">
              <div>
                <h2>➕ Sảnh chờ</h2>
                <p>Thêm người thật, bot, chọn số người tối thiểu rồi bắt đầu.</p>
              </div>
              <button class="btn success" id="startBtn">▶️ Bắt đầu</button>
            </div>
            <div class="form-grid">
              <input id="playerNameInput" placeholder="Tên của bạn" value="Chủ phòng" />
              <input id="joinCodeInput" placeholder="Nhập mã phòng" />
              <input id="joinNameInput" placeholder="Tên người chơi mới" />
              <button class="btn secondary" id="addHumanBtn">Thêm người chơi</button>
            </div>
            <div class="tool-row">
              <label for="minPlayersInput">Số người mục tiêu</label>
              <input id="minPlayersInput" type="range" min="5" max="12" value="8" />
              <b id="minPlayersText">8</b>
              <button class="btn ghost" id="addBotBtn">🤖 Thêm 1 bot</button>
              <button class="btn ghost" id="fillBotsBtn">Tự thêm cho đủ</button>
            </div>
            <h3>✨ Bộ vai trò khi đủ <span id="roleTargetText">8</span> người</h3>
            <div class="role-grid" id="roleGrid"></div>
          </section>

          <section class="card hidden" id="gameCard">
            <div id="winnerBox" class="winner-box hidden"></div>
            <div id="nightPanel" class="hidden">
              <h2>🌙 Pha đêm</h2>
              <div class="action-grid">
                <div class="action-box">
                  <b>💀 Sói chọn cắn</b>
                  <select id="wolfTargetSelect"></select>
                </div>
                <div class="action-box">
                  <b>🛡️ Bảo vệ chọn cứu</b>
                  <select id="doctorTargetSelect"></select>
                </div>
                <div class="action-box">
                  <b>👁️ Tiên tri chọn soi</b>
                  <select id="seerTargetSelect"></select>
                </div>
              </div>
              <label class="checkbox-line">
                <input type="checkbox" id="witchSaveInput" />
                💗 Phù thủy dùng bình cứu nạn nhân đêm nay
              </label>
              <div class="tool-row">
                <button class="btn secondary" id="autoNightBtn">🤖 Tự chọn hành động</button>
                <button class="btn" id="resolveNightBtn">☀️ Kết thúc đêm</button>
              </div>
            </div>
            <div id="dayPanel" class="hidden">
              <h2>☀️ Pha ngày</h2>
              <div class="info-grid">
                <div class="info-box">
                  <b>Thông báo buổi sáng</b>
                  <p id="nightVictimText">Mọi người thức dậy và bắt đầu thảo luận.</p>
                  <p id="seerResultText" class="seer-result"></p>
                </div>
                <div class="info-box">
                  <b>Bỏ phiếu treo cổ</b>
                  <p>Chọn mục tiêu cho từng người còn sống, hoặc để bot tự bỏ phiếu.</p>
                </div>
              </div>
              <div class="vote-list" id="voteList"></div>
              <div class="tool-row">
                <button class="btn secondary" id="autoVoteBtn">🤖 Bot tự bỏ phiếu</button>
                <button class="btn danger" id="resolveVoteBtn">💀 Chốt phiếu</button>
              </div>
            </div>
          </section>

          <section class="card">
            <h2>👥 Danh sách người chơi</h2>
            <div class="player-grid" id="playerGrid"></div>
          </section>
        </div>

        <aside class="side-column">
          <section class="card">
            <h2>👑 Luật thắng</h2>
            <p><b>Phe Làng:</b> thắng khi loại hết Ma Sói.</p>
            <p><b>Phe Sói:</b> thắng khi số Sói còn sống bằng hoặc nhiều hơn số dân làng.</p>
            <p><b>Bot:</b> tự hành động ban đêm và tự bỏ phiếu nếu người chơi chưa chọn.</p>
            <p><b>HTML/Vercel-only:</b> có link để mọi người mở game. Muốn nhiều máy đồng bộ thật thì sau này thêm Firebase/Supabase.</p>
          </section>

          <section class="card">
            <h2>💬 Chat thảo luận</h2>
            <div class="chat-box" id="chatBox"></div>
            <div class="chat-input">
              <input id="chatInput" placeholder="Nhập tin nhắn..." />
              <button class="btn" id="sendChatBtn">Gửi</button>
            </div>
          </section>

          <section class="card">
            <h2>📜 Nhật ký game</h2>
            <div class="log-list" id="logList"></div>
          </section>

          <section class="card">
            <h2>🧪 Kiểm tra</h2>
            <div id="testList"></div>
          </section>
        </aside>
      </div>
    </div>
  </main>

  <script>
    "use strict";

    const VERSION = "v1.0.0";

    const ROLE_DATA = {
      wolf: { name: "Ma sói", team: "Sói", icon: "🐺", desc: "Mỗi đêm cùng bầy sói chọn một người để cắn." },
      villager: { name: "Dân làng", team: "Làng", icon: "🧑‍🌾", desc: "Không có kỹ năng ban đêm, thắng bằng suy luận và bỏ phiếu đúng." },
      seer: { name: "Tiên tri", team: "Làng", icon: "🔮", desc: "Mỗi đêm soi một người để biết họ thuộc phe Sói hay không." },
      doctor: { name: "Bảo vệ", team: "Làng", icon: "🛡️", desc: "Mỗi đêm bảo vệ một người khỏi bị sói cắn." },
      witch: { name: "Phù thủy", team: "Làng", icon: "🧪", desc: "Có một bình cứu. Bản đơn giản cho phép cứu nạn nhân đêm đó." },
      hunter: { name: "Thợ săn", team: "Làng", icon: "🏹", desc: "Vai mạnh trong làng. Bản này hiển thị vai trước, chưa xử lý bắn khi chết." },
      cupid: { name: "Thần tình yêu", team: "Làng", icon: "💘", desc: "Vai đặc biệt. Bản này hiển thị vai trước, chưa xử lý ghép đôi." },
      guard: { name: "Hiệp sĩ", team: "Làng", icon: "⚔️", desc: "Có tiếng nói mạnh khi tranh luận. Bản này tính phiếu như người thường." },
      alphaWolf: { name: "Sói đầu đàn", team: "Sói", icon: "👑🐺", desc: "Sói đặc biệt, cùng phe Sói đi cắn mỗi đêm." }
    };

    const BOT_NAMES = ["An Bot", "Bình Bot", "Chi Bot", "Dũng Bot", "Em Bot", "Giang Bot", "Hạ Bot", "Khoa Bot", "Linh Bot", "Minh Bot", "Nhi Bot", "Phong Bot", "Quân Bot", "Trang Bot", "Vy Bot", "Long Bot", "Tú Bot", "Huy Bot"];

    const ROLE_SET_BY_SIZE = {
      5: ["wolf", "seer", "doctor", "villager", "villager"],
      6: ["wolf", "seer", "doctor", "hunter", "villager", "villager"],
      7: ["wolf", "wolf", "seer", "doctor", "hunter", "villager", "villager"],
      8: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "villager", "villager"],
      9: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"],
      10: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"],
      11: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager"],
      12: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager", "villager"]
    };

    const state = {
      roomCode: makeCode(),
      playerName: "Chủ phòng",
      joinCode: "",
      minPlayers: 8,
      players: [createPlayer("Chủ phòng", false, true)],
      phase: "lobby",
      day: 0,
      selectedWolfTarget: "",
      selectedDoctorTarget: "",
      selectedSeerTarget: "",
      selectedWitchSave: false,
      seerResult: "",
      nightVictim: "",
      votes: {},
      log: ["Phòng đã được tạo. Gửi mã phòng cho bạn bè để vào chơi."],
      chat: [{ from: "Hệ thống", text: "Chào mừng đến với Ma Sói!" }],
      copied: false,
      tests: []
    };

    const els = {};

    document.addEventListener("DOMContentLoaded", init);

    function init() {
      cacheElements();
      bindEvents();
      runSelfTests();
      render();
    }

    function cacheElements() {
      const ids = [
        "resetBtn", "copyBtn", "roomCodeText", "playerCountText", "phaseText", "statusText", "lobbyCard", "gameCard", "winnerBox", "nightPanel", "dayPanel", "playerNameInput", "joinCodeInput", "joinNameInput", "addHumanBtn", "minPlayersInput", "minPlayersText", "roleTargetText", "addBotBtn", "fillBotsBtn", "startBtn", "roleGrid", "wolfTargetSelect", "doctorTargetSelect", "seerTargetSelect", "witchSaveInput", "autoNightBtn", "resolveNightBtn", "nightVictimText", "seerResultText", "voteList", "autoVoteBtn", "resolveVoteBtn", "playerGrid", "chatBox", "chatInput", "sendChatBtn", "logList", "testList"
      ];

      ids.forEach((id) => {
        els[id] = document.getElementById(id);
      });
    }

    function bindEvents() {
      els.resetBtn.addEventListener("click", resetGame);
      els.copyBtn.addEventListener("click", copyInvite);
      els.addHumanBtn.addEventListener("click", addHuman);
      els.addBotBtn.addEventListener("click", () => addBot(1));
      els.fillBotsBtn.addEventListener("click", fillBots);
      els.startBtn.addEventListener("click", startGame);
      els.autoNightBtn.addEventListener("click", autoNightChoices);
      els.resolveNightBtn.addEventListener("click", resolveNight);
      els.autoVoteBtn.addEventListener("click", autoVote);
      els.resolveVoteBtn.addEventListener("click", resolveVote);
      els.sendChatBtn.addEventListener("click", sendChat);
      els.chatInput.addEventListener("keydown", (event) => {
        if (event.key === "Enter") sendChat();
      });
      els.minPlayersInput.addEventListener("input", () => {
        state.minPlayers = Number(els.minPlayersInput.value);
        render();
      });
      els.playerNameInput.addEventListener("input", () => {
        state.playerName = els.playerNameInput.value || "Người chơi";
      });
      els.joinCodeInput.addEventListener("input", () => {
        state.joinCode = els.joinCodeInput.value.toUpperCase();
        els.joinCodeInput.value = state.joinCode;
      });
      els.witchSaveInput.addEventListener("change", () => {
        state.selectedWitchSave = els.witchSaveInput.checked;
      });
      els.wolfTargetSelect.addEventListener("change", () => {
        state.selectedWolfTarget = els.wolfTargetSelect.value;
      });
      els.doctorTargetSelect.addEventListener("change", () => {
        state.selectedDoctorTarget = els.doctorTargetSelect.value;
      });
      els.seerTargetSelect.addEventListener("change", () => {
        state.selectedSeerTarget = els.seerTargetSelect.value;
      });
    }

    function makeCode() {
      return Math.random().toString(36).slice(2, 6).toUpperCase() + "-" + Math.random().toString(36).slice(2, 6).toUpperCase();
    }

    function makeId() {
      if (window.crypto && typeof window.crypto.randomUUID === "function") {
        return window.crypto.randomUUID();
      }
      return "id-" + Date.now() + "-" + Math.random().toString(36).slice(2);
    }

    function shuffle(arr) {
      return arr.slice().sort(() => Math.random() - 0.5);
    }

    function chooseRandom(list) {
      if (!Array.isArray(list) || list.length === 0) return null;
      return list[Math.floor(Math.random() * list.length)];
    }

    function getRoleSet(count) {
      const safeCount = Math.max(5, Math.min(12, Number(count) || 5));
      return ROLE_SET_BY_SIZE[safeCount] || ROLE_SET_BY_SIZE[12];
    }

    function createPlayer(name, isBot, host) {
      return {
        id: makeId(),
        name: name,
        isBot: Boolean(isBot),
        host: Boolean(host),
        alive: true,
        role: null
      };
    }

    function isWolfRole(role) {
      return role === "wolf" || role === "alphaWolf";
    }

    function alivePlayers() {
      return state.players.filter((player) => player.alive);
    }

    function wolvesAlive() {
      return state.players.filter((player) => player.alive && isWolfRole(player.role));
    }

    function villagersAlive() {
      return state.players.filter((player) => player.alive && !isWolfRole(player.role));
    }

    function getWinner() {
      if (state.phase === "lobby") return "";
      if (wolvesAlive().length === 0) return "Làng thắng! Tất cả ma sói đã bị loại.";
      if (wolvesAlive().length >= villagersAlive().length) return "Sói thắng! Số sói đã bằng hoặc vượt số dân làng.";
      return "";
    }

    function pushLog(text) {
      state.log.unshift(text);
      state.log = state.log.slice(0, 14);
      render();
    }

    function resetGame() {
      const hostName = els.playerNameInput.value.trim() || "Chủ phòng";
      state.roomCode = makeCode();
      state.playerName = hostName;
      state.joinCode = "";
      state.players = [createPlayer(hostName, false, true)];
      state.phase = "lobby";
      state.day = 0;
      state.selectedWolfTarget = "";
      state.selectedDoctorTarget = "";
      state.selectedSeerTarget = "";
      state.selectedWitchSave = false;
      state.seerResult = "";
      state.nightVictim = "";
      state.votes = {};
      state.log = ["Phòng mới đã được tạo."];
      state.chat = [{ from: "Hệ thống", text: "Phòng mới đã sẵn sàng." }];
      els.joinCodeInput.value = "";
      render();
    }

    async function copyInvite() {
      const invite = "Mời bạn vào chơi Ma Sói! Mã phòng: " + state.roomCode + ". Mở cùng link web rồi nhập mã để tham gia.";
      try {
        if (navigator.clipboard && navigator.clipboard.writeText) {
          await navigator.clipboard.writeText(invite);
          state.copied = true;
          render();
          setTimeout(() => {
            state.copied = false;
            render();
          }, 1200);
        } else {
          pushLog("Trình duyệt không hỗ trợ copy tự động. Mã phòng là " + state.roomCode + ".");
        }
      } catch (error) {
        pushLog("Không copy tự động được. Mã phòng là " + state.roomCode + ".");
      }
    }

    function addHuman() {
      const name = els.joinNameInput.value.trim();
      if (!name) {
        pushLog("Hãy nhập tên người chơi trước khi thêm.");
        return;
      }
      if (state.players.length >= 12) {
        pushLog("Phòng đã đủ 12 người.");
        return;
      }
      state.players.push(createPlayer(name, false, false));
      els.joinNameInput.value = "";
      pushLog(name + " đã vào phòng bằng mã " + (state.joinCode || state.roomCode) + ".");
    }

    function addBot(count) {
      if (state.players.length >= 12) {
        pushLog("Phòng đã đủ 12 người, không thể thêm bot.");
        return;
      }

      const amount = Math.min(Number(count) || 1, 12 - state.players.length);
      const used = new Set(state.players.map((player) => player.name));

      for (let i = 0; i < amount; i++) {
        const botName = BOT_NAMES.find((name) => !used.has(name)) || "Bot " + (state.players.length + 1);
        used.add(botName);
        state.players.push(createPlayer(botName, true, false));
      }

      pushLog("Đã thêm " + amount + " người máy vào phòng.");
    }

    function fillBots() {
      const need = Math.max(0, state.minPlayers - state.players.length);
      if (need > 0) addBot(need);
      else pushLog("Số người đã đủ, không cần thêm bot.");
    }

    function startGame() {
      let roster = state.players.slice();
      const need = Math.max(0, state.minPlayers - roster.length);

      for (let i = 0; i < need; i++) {
        const botName = BOT_NAMES.find((name) => !roster.some((player) => player.name === name)) || "Bot " + (roster.length + 1);
        roster.push(createPlayer(botName, true, false));
      }

      roster = roster.slice(0, 12);
      const roles = shuffle(getRoleSet(roster.length));
      const shuffledRoster = shuffle(roster);

      state.players = shuffledRoster.map((player, index) => ({
        ...player,
        role: roles[index],
        alive: true
      }));
      state.phase = "night";
      state.day = 1;
      state.votes = {};
      state.nightVictim = "";
      state.seerResult = "";
      state.selectedWolfTarget = "";
      state.selectedDoctorTarget = "";
      state.selectedSeerTarget = "";
      state.selectedWitchSave = false;
      state.log.unshift("Game bắt đầu với " + state.players.length + " người. Đêm 1 bắt đầu.");
      state.log = state.log.slice(0, 14);
      render();
    }

    function autoNightChoices() {
      const alive = alivePlayers();
      const possibleVictims = alive.filter((player) => !isWolfRole(player.role));
      const wolfTarget = state.selectedWolfTarget || (chooseRandom(possibleVictims) || {}).id || "";
      const doctorTarget = state.selectedDoctorTarget || (chooseRandom(alive) || {}).id || "";
      const seerTarget = state.selectedSeerTarget || (chooseRandom(alive.filter((player) => player.role !== "seer")) || {}).id || "";

      state.selectedWolfTarget = wolfTarget;
      state.selectedDoctorTarget = doctorTarget;
      state.selectedSeerTarget = seerTarget;
      pushLog("Bot và các vai chưa chọn đã tự động hành động trong đêm.");
    }

    function resolveNight() {
      const wolfTarget = state.players.find((player) => player.id === state.selectedWolfTarget);
      const doctorTarget = state.players.find((player) => player.id === state.selectedDoctorTarget);
      const seerTarget = state.players.find((player) => player.id === state.selectedSeerTarget);
      let victim = null;

      if (wolfTarget) {
        const savedByDoctor = doctorTarget && doctorTarget.id === wolfTarget.id;
        const savedByWitch = state.selectedWitchSave;
        if (!savedByDoctor && !savedByWitch) {
          victim = wolfTarget;
          state.players = state.players.map((player) => player.id === wolfTarget.id ? { ...player, alive: false } : player);
        }
      }

      if (seerTarget) {
        const isWolf = isWolfRole(seerTarget.role);
        state.seerResult = seerTarget.name + " " + (isWolf ? "thuộc phe Sói" : "không thuộc phe Sói") + ".";
      } else {
        state.seerResult = "";
      }

      state.nightVictim = victim ? victim.name + " đã bị sói cắn." : "Không ai chết trong đêm nay.";
      state.phase = "day";
      state.votes = {};
      state.log.unshift(victim ? "Sáng ngày " + state.day + ": " + victim.name + " đã chết." : "Sáng ngày " + state.day + ": không ai chết.");
      state.log = state.log.slice(0, 14);
      render();
    }

    function autoVote() {
      const alive = alivePlayers();
      alive.forEach((voter) => {
        if (!state.votes[voter.id]) {
          const candidates = alive.filter((player) => player.id !== voter.id);
          const suspicious = candidates.filter((player) => isWolfRole(player.role));
          const pool = voter.isBot && Math.random() > 0.45 ? candidates : (suspicious.length ? suspicious : candidates);
          const pick = chooseRandom(pool);
          if (pick) state.votes[voter.id] = pick.id;
        }
      });
      pushLog("Các phiếu còn thiếu đã được tự động bỏ.");
    }

    function resolveVote() {
      const count = {};
      Object.values(state.votes).forEach((targetId) => {
        count[targetId] = (count[targetId] || 0) + 1;
      });
      const sorted = Object.entries(count).sort((a, b) => b[1] - a[1]);

      if (!sorted.length) {
        state.phase = "night";
        state.day += 1;
        pushLog("Không đủ phiếu, không ai bị treo cổ. Đêm tiếp theo bắt đầu.");
        return;
      }

      const targetId = sorted[0][0];
      const amount = sorted[0][1];
      const eliminated = state.players.find((player) => player.id === targetId);
      state.players = state.players.map((player) => player.id === targetId ? { ...player, alive: false } : player);
      state.phase = "night";
      state.day += 1;
      state.selectedWolfTarget = "";
      state.selectedDoctorTarget = "";
      state.selectedSeerTarget = "";
      state.selectedWitchSave = false;
      state.nightVictim = "";
      state.votes = {};
      pushLog((eliminated ? eliminated.name : "Một người") + " bị treo cổ với " + amount + " phiếu. Đêm tiếp theo bắt đầu.");
    }

    function sendChat() {
      const text = els.chatInput.value.trim();
      if (!text) return;
      state.chat.push({ from: els.playerNameInput.value.trim() || "Người chơi", text: text });
      state.chat = state.chat.slice(-8);
      els.chatInput.value = "";
      render();
    }

    function render() {
      const winner = getWinner();
      els.roomCodeText.textContent = state.roomCode;
      els.playerCountText.textContent = state.players.length + "/" + state.minPlayers;
      els.phaseText.textContent = state.phase === "lobby" ? "Sảnh" : state.phase === "night" ? "Đêm " + state.day : "Ngày " + state.day;
      els.statusText.textContent = winner || "Game tiếp tục cho đến khi Làng hoặc Sói thắng.";
      els.copyBtn.textContent = state.copied ? "✅ Đã copy" : "📋 Copy lời mời";
      els.minPlayersInput.value = String(state.minPlayers);
      els.minPlayersText.textContent = String(state.minPlayers);
      els.roleTargetText.textContent = String(state.minPlayers);
      els.witchSaveInput.checked = state.selectedWitchSave;

      toggle(els.lobbyCard, state.phase !== "lobby");
      toggle(els.gameCard, state.phase === "lobby");
      toggle(els.winnerBox, !winner);
      toggle(els.nightPanel, state.phase !== "night" || Boolean(winner));
      toggle(els.dayPanel, state.phase !== "day" || Boolean(winner));

      if (winner) {
        els.winnerBox.innerHTML = "<h2>" + escapeHtml(winner) + "</h2><p>Bấm “Phòng mới” để chơi ván khác.</p>";
      }

      renderRoles();
      renderNightSelects();
      renderVoteList();
      renderPlayers();
      renderChat();
      renderLog();
      renderTests();

      els.nightVictimText.textContent = state.nightVictim || "Mọi người thức dậy và bắt đầu thảo luận.";
      els.seerResultText.textContent = state.seerResult ? "Kết quả riêng của Tiên tri: " + state.seerResult : "";
    }

    function renderRoles() {
      const counts = {};
      getRoleSet(state.minPlayers).forEach((role) => {
        counts[role] = (counts[role] || 0) + 1;
      });
      els.roleGrid.innerHTML = Object.keys(counts).map((role) => {
        const data = ROLE_DATA[role];
        return "<div class=\"role-mini\"><span>" + data.icon + "</span><b>" + escapeHtml(data.name) + " x" + counts[role] + "</b><small>Phe " + escapeHtml(data.team) + "</small></div>";
      }).join("");
    }

    function renderNightSelects() {
      const alive = alivePlayers();
      const victimOptions = alive.filter((player) => !isWolfRole(player.role));
      fillSelect(els.wolfTargetSelect, victimOptions, state.selectedWolfTarget, "Chưa chọn");
      fillSelect(els.doctorTargetSelect, alive, state.selectedDoctorTarget, "Chưa chọn");
      fillSelect(els.seerTargetSelect, alive, state.selectedSeerTarget, "Chưa chọn");
    }

    function fillSelect(selectEl, players, selectedValue, emptyLabel) {
      const html = ["<option value=\"\">" + escapeHtml(emptyLabel) + "</option>"];
      players.forEach((player) => {
        html.push("<option value=\"" + escapeHtml(player.id) + "\">" + escapeHtml(player.name) + "</option>");
      });
      selectEl.innerHTML = html.join("");
      selectEl.value = selectedValue;
    }

    function renderVoteList() {
      const alive = alivePlayers();
      els.voteList.innerHTML = "";
      alive.forEach((voter) => {
        const row = document.createElement("div");
        row.className = "vote-row";

        const label = document.createElement("b");
        label.textContent = voter.name + (voter.isBot ? " BOT" : "");

        const select = document.createElement("select");
        select.innerHTML = "<option value=\"\">Chọn người bị nghi</option>";
        alive.filter((player) => player.id !== voter.id).forEach((candidate) => {
          const option = document.createElement("option");
          option.value = candidate.id;
          option.textContent = candidate.name;
          select.appendChild(option);
        });
        select.value = state.votes[voter.id] || "";
        select.addEventListener("change", () => {
          state.votes[voter.id] = select.value;
        });

        row.appendChild(label);
        row.appendChild(select);
        els.voteList.appendChild(row);
      });
    }

    function renderPlayers() {
      els.playerGrid.innerHTML = state.players.map((player) => {
        const role = player.role ? ROLE_DATA[player.role] : null;
        const status = (player.host ? "Chủ phòng " : "") + (player.isBot ? "BOT " : "") + (player.alive ? "Còn sống" : "Đã chết");
        const roleBox = role ? "<div class=\"role-box\"><b>" + escapeHtml(role.name) + "</b><small>Phe " + escapeHtml(role.team) + "</small><p>" + escapeHtml(role.desc) + "</p></div>" : "";
        return "<div class=\"player-card " + (player.alive ? "" : "dead") + "\"><div class=\"player-top\"><div><b>" + escapeHtml(player.name) + "</b><small>" + escapeHtml(status) + "</small></div><span>" + (role ? role.icon : player.isBot ? "🤖" : "🙂") + "</span></div>" + roleBox + "</div>";
      }).join("");
    }

    function renderChat() {
      els.chatBox.innerHTML = state.chat.map((message) => "<p><b>" + escapeHtml(message.from) + ":</b> " + escapeHtml(message.text) + "</p>").join("");
      els.chatBox.scrollTop = els.chatBox.scrollHeight;
    }

    function renderLog() {
      els.logList.innerHTML = state.log.map((entry) => "<div>" + escapeHtml(entry) + "</div>").join("");
    }

    function renderTests() {
      els.testList.innerHTML = state.tests.map((test) => {
        const symbol = test.pass ? "✅" : "❌";
        return "<div class=\"test-row\">" + symbol + " " + escapeHtml(test.name) + "</div>";
      }).join("");
    }

    function toggle(element, shouldHide) {
      element.classList.toggle("hidden", shouldHide);
    }

    function escapeHtml(value) {
      return String(value)
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
    }

    function runSelfTests() {
      const tests = [
        { name: "Phiên bản đúng v1.0.0", pass: VERSION === "v1.0.0" },
        { name: "Bộ 5 người có đúng 5 vai", pass: getRoleSet(5).length === 5 },
        { name: "Bộ 8 người có Sói đầu đàn", pass: getRoleSet(8).includes("alphaWolf") },
        { name: "Số quá lớn được giới hạn về 12 vai", pass: getRoleSet(99).length === 12 },
        { name: "Chọn ngẫu nhiên danh sách rỗng trả về null", pass: chooseRandom([]) === null },
        { name: "Người chơi mới mặc định còn sống", pass: createPlayer("Test", false, false).alive === true },
        { name: "Có đủ DOM chính để render", pass: Boolean(els.playerGrid && els.roleGrid && els.gameCard) }
      ];
      state.tests = tests;
      tests.forEach((test) => console.assert(test.pass, "Self-test failed: " + test.name));
    }
  </script>
</body>
</html>    name: "Bảo vệ",
    team: "Làng",
    icon: "🛡️",
    desc: "Mỗi đêm bảo vệ một người khỏi bị sói cắn.",
  },
  witch: {
    name: "Phù thủy",
    team: "Làng",
    icon: "🧪",
    desc: "Có một bình cứu. Bản đơn giản cho phép cứu nạn nhân đêm đó.",
  },
  hunter: {
    name: "Thợ săn",
    team: "Làng",
    icon: "🏹",
    desc: "Khi chết có thể kéo theo một người. Bản này để vai trò hiển thị trước.",
  },
  cupid: {
    name: "Thần tình yêu",
    team: "Làng",
    icon: "💘",
    desc: "Đêm đầu ghép đôi hai người. Bản này để vai trò hiển thị trước.",
  },
  guard: {
    name: "Hiệp sĩ",
    team: "Làng",
    icon: "⚔️",
    desc: "Có tiếng nói mạnh khi tranh luận. Bản này tính phiếu như người thường.",
  },
  alphaWolf: {
    name: "Sói đầu đàn",
    team: "Sói",
    icon: "👑🐺",
    desc: "Sói đặc biệt, cùng phe Sói đi cắn mỗi đêm.",
  },
};

const BOT_NAMES = [
  "An Bot",
  "Bình Bot",
  "Chi Bot",
  "Dũng Bot",
  "Em Bot",
  "Giang Bot",
  "Hạ Bot",
  "Khoa Bot",
  "Linh Bot",
  "Minh Bot",
  "Nhi Bot",
  "Phong Bot",
  "Quân Bot",
  "Trang Bot",
  "Vy Bot",
  "Long Bot",
  "Tú Bot",
  "Huy Bot",
];

const ROLE_SET_BY_SIZE = {
  5: ["wolf", "seer", "doctor", "villager", "villager"],
  6: ["wolf", "seer", "doctor", "hunter", "villager", "villager"],
  7: ["wolf", "wolf", "seer", "doctor", "hunter", "villager", "villager"],
  8: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "villager", "villager"],
  9: ["alphaWolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"],
  10: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "villager", "villager"],
  11: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager"],
  12: ["alphaWolf", "wolf", "wolf", "seer", "doctor", "witch", "hunter", "cupid", "guard", "villager", "villager", "villager"],
};

function makeCode() {
  return Math.random().toString(36).slice(2, 6).toUpperCase() + "-" + Math.random().toString(36).slice(2, 6).toUpperCase();
}

function shuffle(arr) {
  return [...arr].sort(() => Math.random() - 0.5);
}

function chooseRandom(list) {
  if (!list.length) return null;
  return list[Math.floor(Math.random() * list.length)];
}

function getRoleSet(count) {
  const safeCount = Math.max(5, Math.min(12, count));
  return ROLE_SET_BY_SIZE[safeCount] || ROLE_SET_BY_SIZE[12];
}

function createPlayer(name, isBot = false, host = false) {
  return {
    id: crypto.randomUUID(),
    name,
    isBot,
    host,
    alive: true,
    role: null,
  };
}

function Button({ children, onClick, variant = "primary", disabled = false }) {
  return (
    <button className={`btn ${variant}`} onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

function Card({ children, className = "" }) {
  return <section className={`card ${className}`}>{children}</section>;
}

export default function App() {
  const [roomCode, setRoomCode] = useState(makeCode());
  const [playerName, setPlayerName] = useState("Chủ phòng");
  const [joinName, setJoinName] = useState("");
  const [joinCode, setJoinCode] = useState("");
  const [minPlayers, setMinPlayers] = useState(8);
  const [players, setPlayers] = useState([createPlayer("Chủ phòng", false, true)]);
  const [phase, setPhase] = useState("lobby");
  const [day, setDay] = useState(0);
  const [selectedWolfTarget, setSelectedWolfTarget] = useState("");
  const [selectedDoctorTarget, setSelectedDoctorTarget] = useState("");
  const [selectedSeerTarget, setSelectedSeerTarget] = useState("");
  const [selectedWitchSave, setSelectedWitchSave] = useState(false);
  const [seerResult, setSeerResult] = useState(null);
  const [nightVictim, setNightVictim] = useState(null);
  const [votes, setVotes] = useState({});
  const [log, setLog] = useState(["Phòng đã được tạo. Gửi mã phòng cho bạn bè để vào chơi."]);
  const [chat, setChat] = useState([{ from: "Hệ thống", text: "Chào mừng đến với Ma Sói!" }]);
  const [chatText, setChatText] = useState("");
  const [copied, setCopied] = useState(false);

  const alivePlayers = players.filter((p) => p.alive);
  const wolves = players.filter((p) => p.alive && ["wolf", "alphaWolf"].includes(p.role));
  const villagers = players.filter((p) => p.alive && !["wolf", "alphaWolf"].includes(p.role));

  const winner = useMemo(() => {
    if (phase === "lobby") return null;
    if (wolves.length === 0) return "Làng thắng! Tất cả ma sói đã bị loại.";
    if (wolves.length >= villagers.length) return "Sói thắng! Số sói đã bằng hoặc vượt số dân làng.";
    return null;
  }, [wolves.length, villagers.length, phase]);

  const roleSummary = useMemo(() => {
    const set = getRoleSet(minPlayers);
    const counted = set.reduce((acc, role) => {
      acc[role] = (acc[role] || 0) + 1;
      return acc;
    }, {});
    return Object.entries(counted);
  }, [minPlayers]);

  function pushLog(text) {
    setLog((prev) => [text, ...prev].slice(0, 14));
  }

  function resetGame() {
    setRoomCode(makeCode());
    setPlayers([createPlayer(playerName || "Chủ phòng", false, true)]);
    setPhase("lobby");
    setDay(0);
    setSelectedWolfTarget("");
    setSelectedDoctorTarget("");
    setSelectedSeerTarget("");
    setSelectedWitchSave(false);
    setSeerResult(null);
    setNightVictim(null);
    setVotes({});
    setLog(["Phòng mới đã được tạo."]);
    setChat([{ from: "Hệ thống", text: "Phòng mới đã sẵn sàng." }]);
  }

  function copyInvite() {
    const invite = `Mời bạn vào chơi Ma Sói! Mã phòng: ${roomCode}. Mở cùng link web rồi nhập mã để tham gia.`;
    navigator.clipboard?.writeText(invite);
    setCopied(true);
    setTimeout(() => setCopied(false), 1200);
  }

  function addHuman() {
    const name = joinName.trim();
    if (!name) return;
    setPlayers((prev) => [...prev, createPlayer(name, false)].slice(0, 12));
    setJoinName("");
    pushLog(`${name} đã vào phòng bằng mã ${joinCode || roomCode}.`);
  }

  function addBot(count = 1) {
    setPlayers((prev) => {
      const used = new Set(prev.map((p) => p.name));
      const fresh = [];
      for (let i = 0; i < count; i++) {
        const name = BOT_NAMES.find((n) => !used.has(n)) || `Bot ${prev.length + i + 1}`;
        used.add(name);
        fresh.push(createPlayer(name, true));
      }
      return [...prev, ...fresh].slice(0, 12);
    });
    pushLog(`Đã thêm ${count} người máy vào phòng.`);
  }

  function fillBots() {
    const need = Math.max(0, minPlayers - players.length);
    if (need > 0) addBot(need);
    else pushLog("Số người đã đủ, không cần thêm bot.");
  }

  function startGame() {
    let roster = [...players];
    const need = Math.max(0, minPlayers - roster.length);

    for (let i = 0; i < need; i++) {
      const name = BOT_NAMES.find((n) => !roster.some((p) => p.name === n)) || `Bot ${roster.length + 1}`;
      roster.push(createPlayer(name, true));
    }

    roster = roster.slice(0, 12);
    const roles = shuffle(getRoleSet(roster.length));
    const assigned = shuffle(roster).map((p, i) => ({
      ...p,
      role: roles[i],
      alive: true,
    }));

    setPlayers(assigned);
    setPhase("night");
    setDay(1);
    setVotes({});
    setNightVictim(null);
    setSeerResult(null);
    pushLog(`Game bắt đầu với ${assigned.length} người. Đêm 1 bắt đầu.`);
  }

  function autoNightChoices() {
    const possibleVictims = alivePlayers.filter((p) => !["wolf", "alphaWolf"].includes(p.role));
    const wolfTarget = selectedWolfTarget || chooseRandom(possibleVictims)?.id || "";
    const doctorTarget = selectedDoctorTarget || chooseRandom(alivePlayers)?.id || "";
    const seerTarget = selectedSeerTarget || chooseRandom(alivePlayers.filter((p) => p.role !== "seer"))?.id || "";

    setSelectedWolfTarget(wolfTarget);
    setSelectedDoctorTarget(doctorTarget);
    setSelectedSeerTarget(seerTarget);
    pushLog("Bot và các vai chưa chọn đã tự động hành động trong đêm.");
  }

  function resolveNight() {
    const wolfTarget = players.find((p) => p.id === selectedWolfTarget);
    const doctorTarget = players.find((p) => p.id === selectedDoctorTarget);
    const seerTarget = players.find((p) => p.id === selectedSeerTarget);

    let updated = players;
    let victim = null;

    if (wolfTarget) {
      const savedByDoctor = doctorTarget?.id === wolfTarget.id;
      const savedByWitch = selectedWitchSave;

      if (!savedByDoctor && !savedByWitch) {
        victim = wolfTarget;
        updated = players.map((p) => (p.id === wolfTarget.id ? { ...p, alive: false } : p));
      }
    }

    if (seerTarget) {
      const isWolf = ["wolf", "alphaWolf"].includes(seerTarget.role);
      setSeerResult(`${seerTarget.name} ${isWolf ? "thuộc phe Sói" : "không thuộc phe Sói"}.`);
    }

    setPlayers(updated);
    setNightVictim(victim ? `${victim.name} đã bị sói cắn.` : "Không ai chết trong đêm nay.");
    setPhase("day");
    setVotes({});
    pushLog(victim ? `Sáng ngày ${day}: ${victim.name} đã chết.` : `Sáng ngày ${day}: không ai chết.`);
  }

  function castVote(voterId, targetId) {
    setVotes((prev) => ({ ...prev, [voterId]: targetId }));
  }

  function autoVote() {
    const nextVotes = { ...votes };

    alivePlayers.forEach((voter) => {
      if (!nextVotes[voter.id]) {
        const candidates = alivePlayers.filter((p) => p.id !== voter.id);
        const suspicious = candidates.filter((p) => ["wolf", "alphaWolf"].includes(p.role));
        const pick = voter.isBot && Math.random() > 0.45 ? chooseRandom(candidates) : chooseRandom(suspicious.length ? suspicious : candidates);
        if (pick) nextVotes[voter.id] = pick.id;
      }
    });

    setVotes(nextVotes);
    pushLog("Các phiếu còn thiếu đã được tự động bỏ.");
  }

  function resolveVote() {
    const count = {};
    Object.values(votes).forEach((targetId) => {
      count[targetId] = (count[targetId] || 0) + 1;
    });

    const sorted = Object.entries(count).sort((a, b) => b[1] - a[1]);

    if (!sorted.length) {
      pushLog("Không đủ phiếu, không ai bị treo cổ.");
      setPhase("night");
      setDay((d) => d + 1);
      return;
    }

    const [targetId, amount] = sorted[0];
    const eliminated = players.find((p) => p.id === targetId);

    setPlayers((prev) => prev.map((p) => (p.id === targetId ? { ...p, alive: false } : p)));
    setPhase("night");
    setDay((d) => d + 1);
    setSelectedWolfTarget("");
    setSelectedDoctorTarget("");
    setSelectedSeerTarget("");
    setSelectedWitchSave(false);
    setNightVictim(null);
    pushLog(`${eliminated?.name || "Một người"} bị treo cổ với ${amount} phiếu. Đêm tiếp theo bắt đầu.`);
  }

  function sendChat() {
    const text = chatText.trim();
    if (!text) return;
    setChat((prev) => [...prev.slice(-7), { from: playerName || "Người chơi", text }]);
    setChatText("");
  }

  return (
    <main className="page">
      <div className="container">
        <motion.header initial={{ opacity: 0, y: -16 }} animate={{ opacity: 1, y: 0 }} className="hero">
          <div>
            <div className="badge"><Moon size={16} /> Ma Sói Web Demo</div>
            <h1>Trò chơi Ma Sói nhiều vai trò</h1>
            <p>Tạo phòng, chia mã mời bạn bè, tự thêm bot khi thiếu người và chơi theo vòng Đêm - Ngày - Bỏ phiếu.</p>
          </div>
          <div className="heroActions">
            <Button onClick={resetGame} variant="secondary"><RefreshCcw size={16} /> Phòng mới</Button>
            <Button onClick={copyInvite}><Copy size={16} /> {copied ? "Đã copy" : "Copy lời mời"}</Button>
          </div>
        </motion.header>

        <div className="layout">
          <div className="mainColumn">
            <Card>
              <div className="statsGrid">
                <div className="statBox">
                  <span><Wifi size={16} /> Mã phòng</span>
                  <strong>{roomCode}</strong>
                  <small>Gửi mã này cho bạn bè. Bản Vercel-only chỉ mô phỏng nhập phòng trên cùng trình duyệt.</small>
                </div>
                <div className="statBox">
                  <span><Users size={16} /> Người chơi</span>
                  <strong>{players.length}/{minPlayers}</strong>
                  <small>Tối thiểu 5, tối đa 12. Thiếu người thì game tự thêm bot.</small>
                </div>
                <div className="statBox">
                  <span>{phase === "night" ? <Moon size={16} /> : <Sun size={16} />} Trạng thái</span>
                  <strong>{phase === "lobby" ? "Sảnh" : phase === "night" ? `Đêm ${day}` : `Ngày ${day}`}</strong>
                  <small>{winner || "Game tiếp tục cho đến khi Làng hoặc Sói thắng."}</small>
                </div>
              </div>
            </Card>

            {phase === "lobby" && (
              <Card>
                <div className="sectionHeader">
                  <div>
                    <h2><UserPlus size={24} /> Sảnh chờ</h2>
                    <p>Thêm người thật, bot, chọn số người tối thiểu rồi bắt đầu.</p>
                  </div>
                  <Button onClick={startGame} variant="success"><Play size={16} /> Bắt đầu</Button>
                </div>

                <div className="formGrid">
                  <input value={playerName} onChange={(e) => setPlayerName(e.target.value)} placeholder="Tên của bạn" />
                  <input value={joinCode} onChange={(e) => setJoinCode(e.target.value.toUpperCase())} placeholder="Nhập mã phòng" />
                  <input value={joinName} onChange={(e) => setJoinName(e.target.value)} placeholder="Tên người chơi mới" />
                  <Button onClick={addHuman} variant="secondary">Thêm người chơi</Button>
                </div>

                <div className="toolRow">
                  <label>Số người mục tiêu</label>
                  <input type="range" min="5" max="12" value={minPlayers} onChange={(e) => setMinPlayers(Number(e.target.value))} />
                  <b>{minPlayers}</b>
                  <Button onClick={() => addBot(1)} variant="ghost"><Bot size={16} /> Thêm 1 bot</Button>
                  <Button onClick={fillBots} variant="ghost">Tự thêm cho đủ</Button>
                </div>

                <h3 className="miniTitle"><Sparkles size={18} /> Bộ vai trò khi đủ {minPlayers} người</h3>
                <div className="roleGrid">
                  {roleSummary.map(([role, count]) => (
                    <div key={role} className="roleMini">
                      <span>{ROLE_DATA[role].icon}</span>
                      <b>{ROLE_DATA[role].name} x{count}</b>
                      <small>Phe {ROLE_DATA[role].team}</small>
                    </div>
                  ))}
                </div>
              </Card>
            )}

            {phase !== "lobby" && (
              <Card>
                <AnimatePresence mode="wait">
                  {winner ? (
                    <motion.div key="winner" initial={{ opacity: 0, scale: 0.96 }} animate={{ opacity: 1, scale: 1 }} className="winnerBox">
                      <h2>{winner}</h2>
                      <p>Bấm “Phòng mới” để chơi ván khác.</p>
                    </motion.div>
                  ) : phase === "night" ? (
                    <motion.div key="night" initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }}>
                      <h2><Moon size={24} /> Pha đêm</h2>
                      <div className="actionGrid">
                        <ActionSelect title="Sói chọn cắn" icon={<Skull size={20} />} value={selectedWolfTarget} onChange={setSelectedWolfTarget} players={alivePlayers.filter((p) => !["wolf", "alphaWolf"].includes(p.role))} />
                        <ActionSelect title="Bảo vệ chọn cứu" icon={<Shield size={20} />} value={selectedDoctorTarget} onChange={setSelectedDoctorTarget} players={alivePlayers} />
                        <ActionSelect title="Tiên tri chọn soi" icon={<Eye size={20} />} value={selectedSeerTarget} onChange={setSelectedSeerTarget} players={alivePlayers} />
                      </div>

                      <label className="checkboxLine">
                        <input type="checkbox" checked={selectedWitchSave} onChange={(e) => setSelectedWitchSave(e.target.checked)} />
                        <HeartPulse size={20} /> Phù thủy dùng bình cứu nạn nhân đêm nay
                      </label>

                      <div className="toolRow">
                        <Button onClick={autoNightChoices} variant="secondary"><Bot size={16} /> Tự chọn hành động</Button>
                        <Button onClick={resolveNight}><Sun size={16} /> Kết thúc đêm</Button>
                      </div>
                    </motion.div>
                  ) : (
                    <motion.div key="day" initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }}>
                      <h2><Sun size={24} /> Pha ngày</h2>
                      <div className="infoGrid">
                        <div className="infoBox">
                          <b>Thông báo buổi sáng</b>
                          <p>{nightVictim || "Mọi người thức dậy và bắt đầu thảo luận."}</p>
                          {seerResult && <p className="seerResult">Kết quả riêng của Tiên tri: {seerResult}</p>}
                        </div>
                        <div className="infoBox">
                          <b>Bỏ phiếu treo cổ</b>
                          <p>Chọn mục tiêu cho từng người còn sống, hoặc để bot tự bỏ phiếu.</p>
                        </div>
                      </div>

                      <div className="voteList">
                        {alivePlayers.map((voter) => (
                          <div key={voter.id} className="voteRow">
                            <b>{voter.name} {voter.isBot && <small>BOT</small>}</b>
                            <select value={votes[voter.id] || ""} onChange={(e) => castVote(voter.id, e.target.value)}>
                              <option value="">Chọn người bị nghi</option>
                              {alivePlayers.filter((p) => p.id !== voter.id).map((p) => <option key={p.id} value={p.id}>{p.name}</option>)}
                            </select>
                          </div>
                        ))}
                      </div>

                      <div className="toolRow">
                        <Button onClick={autoVote} variant="secondary"><Bot size={16} /> Bot tự bỏ phiếu</Button>
                        <Button onClick={resolveVote} variant="danger"><Skull size={16} /> Chốt phiếu</Button>
                      </div>
                    </motion.div>
                  )}
                </AnimatePresence>
              </Card>
            )}

            <Card>
              <h2><Users size={24} /> Danh sách người chơi</h2>
              <div className="playerGrid">
                {players.map((p) => (
                  <motion.div layout key={p.id} className={`playerCard ${p.alive ? "" : "dead"}`}>
                    <div className="playerTop">
                      <div>
                        <b>{p.name}</b>
                        <small>{p.host && "Chủ phòng "}{p.isBot && "BOT "}{p.alive ? "Còn sống" : "Đã chết"}</small>
                      </div>
                      <span>{p.role ? ROLE_DATA[p.role].icon : p.isBot ? "🤖" : "🙂"}</span>
                    </div>
                    {p.role && (
                      <div className="roleBox">
                        <b>{ROLE_DATA[p.role].name}</b>
                        <small>Phe {ROLE_DATA[p.role].team}</small>
                        <p>{ROLE_DATA[p.role].desc}</p>
                      </div>
                    )}
                  </motion.div>
                ))}
              </div>
            </Card>
          </div>

          <aside className="sideColumn">
            <Card>
              <h2><Crown size={22} /> Luật thắng</h2>
              <p><b>Phe Làng:</b> thắng khi loại hết Ma Sói.</p>
              <p><b>Phe Sói:</b> thắng khi số Sói còn sống bằng hoặc nhiều hơn số dân làng.</p>
              <p><b>Bot:</b> tự hành động ban đêm và tự bỏ phiếu nếu người chơi chưa chọn.</p>
              <p><b>Vercel-only:</b> có link để mọi người mở game. Muốn nhiều máy đồng bộ thật thì sau này thêm Firebase/Supabase.</p>
            </Card>

            <Card>
              <h2><MessageCircle size={22} /> Chat thảo luận</h2>
              <div className="chatBox">
                {chat.map((m, idx) => (
                  <p key={idx}><b>{m.from}:</b> {m.text}</p>
                ))}
              </div>
              <div className="chatInput">
                <input value={chatText} onChange={(e) => setChatText(e.target.value)} onKeyDown={(e) => e.key === "Enter" && sendChat()} placeholder="Nhập tin nhắn..." />
                <Button onClick={sendChat}>Gửi</Button>
              </div>
            </Card>

            <Card>
              <h2>Nhật ký game</h2>
              <div className="logList">
                {log.map((entry, idx) => <div key={idx}>{entry}</div>)}
              </div>
            </Card>
          </aside>
        </div>
      </div>
    </main>
  );
}

function ActionSelect({ title, icon, value, onChange, players }) {
  return (
    <div className="actionBox">
      <b>{icon} {title}</b>
      <select value={value} onChange={(e) => onChange(e.target.value)}>
        <option value="">Chưa chọn</option>
        {players.map((p) => <option key={p.id} value={p.id}>{p.name}</option>)}
      </select>
    </div>
  );
}
