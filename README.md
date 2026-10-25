هذا هو ملف HTML واحد يحتوي على محرك ألعاب ثنائي الأبعاد كامل يعمل في المتصفح. يتضمن محرر مشاهد، فيزياء، سكربتات JavaScript، وتصدير لعبة مستقلة.

```html
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>GameForge — محرك الألعاب</title>
<style>
  :root{
    --bg:#1b1b1f; --panel:#24242a; --panel2:#2c2c34; --border:#38383f;
    --text:#e4e4e8; --muted:#8a8a96; --accent:#4a9eff; --accent2:#7c5cff;
    --green:#3ddc84; --red:#ff5c5c; --yellow:#ffc857;
  }
  *{box-sizing:border-box;margin:0;padding:0}
  html,body{height:100%}
  body{
    font-family:'Segoe UI',Tahoma,Arial,sans-serif;background:var(--bg);color:var(--text);
    display:flex;flex-direction:column;overflow:hidden;font-size:13px;user-select:none;
  }
  /* ===== Toolbar ===== */
  #toolbar{
    height:50px;display:flex;align-items:center;gap:8px;padding:0 12px;
    background:linear-gradient(180deg,#2a2a32,#232329);border-bottom:1px solid var(--border);
    flex-shrink:0;
  }
  .logo{font-weight:800;font-size:16px;letter-spacing:.5px;
    background:linear-gradient(90deg,var(--accent),var(--accent2));
    -webkit-background-clip:text;background-clip:text;color:transparent;margin-left:6px}
  .sep{width:1px;height:26px;background:var(--border);margin:0 4px}
  .btn{
    background:var(--panel2);border:1px solid var(--border);color:var(--text);
    padding:7px 13px;border-radius:7px;cursor:pointer;font-size:12.5px;
    display:flex;align-items:center;gap:6px;transition:.15s;font-family:inherit;white-space:nowrap;
  }
  .btn:hover{background:#383840;border-color:#4a4a55;transform:translateY(-1px)}
  .btn:active{transform:translateY(0)}
  .btn.play{background:#1f4a2c;border-color:#2d6b40;color:#7dffb0}
  .btn.play:hover{background:#286038}
  .btn.play.on{background:var(--green);color:#08301a;border-color:var(--green);font-weight:700}
  .btn.stop{background:#4a1f1f;border-color:#6b2d2d;color:#ff8a8a}
  .btn.stop:hover{background:#603030}
  .btn.pause{background:#4a3d1f;border-color:#6b5a2d;color:#ffd97d}
  .spacer{flex:1}
  /* ===== Layout ===== */
  #main{flex:1;display:flex;overflow:hidden}
  .panel{background:var(--panel);display:flex;flex-direction:column;overflow:hidden}
  #left{width:250px;border-left:1px solid var(--border);flex-shrink:0}
  #right{width:300px;border-right:1px solid var(--border);flex-shrink:0}
  #center{flex:1;display:flex;flex-direction:column;background:#151519;min-width:0}
  .panel-head{
    height:34px;display:flex;align-items:center;justify-content:space-between;
    padding:0 10px;background:#1f1f25;border-bottom:1px solid var(--border);
    font-size:11.5px;font-weight:700;letter-spacing:.4px;color:var(--muted);
    text-transform:uppercase;flex-shrink:0;
  }
  .panel-body{flex:1;overflow-y:auto;overflow-x:hidden}
  .panel-body::-webkit-scrollbar{width:9px}
  .panel-body::-webkit-scrollbar-thumb{background:#3a3a44;border-radius:5px}
  .panel-body::-webkit-scrollbar-track{background:transparent}
  /* ===== Hierarchy ===== */
  .hi{
    display:flex;align-items:center;gap:7px;padding:7px 10px;cursor:pointer;
    border-right:2px solid transparent;font-size:12.5px;
  }
  .hi:hover{background:#2c2c35}
  .hi.sel{background:#1e3a5c;border-right-color:var(--accent)}
  .hi .ic{font-size:13px;width:16px;text-align:center;opacity:.85}
  .hi .nm{flex:1;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
  .hi .del{opacity:0;color:var(--red);font-size:14px;padding:0 3px;border-radius:4px}
  .hi:hover .del{opacity:1}
  .hi .del:hover{background:#4a1f1f}
  /* ===== Canvas ===== */
  #viewport-wrap{flex:1;position:relative;overflow:hidden}
  #viewport{display:block;cursor:default}
  #viewport.moving{cursor:grabbing}
  #viewport.panning{cursor:move}
  /* ===== Inspector ===== */
  .field{margin:9px 11px}
  .field label{display:block;font-size:10.5px;color:var(--muted);margin-bottom:4px;font-weight:600;letter-spacing:.3px}
  .field input[type=text],.field input[type=number],.field select,.field textarea{
    width:100%;background:#1a1a20;border:1px solid var(--border);color:var(--text);
    padding:6px 9px;border-radius:6px;font-size:12.5px;font-family:inherit;outline:none;
  }
  .field input:focus,.field select:focus,.field textarea:focus{border-color:var(--accent)}
  .field textarea{resize:vertical;min-height:130px;font-family:'Consolas','Courier New',monospace;
    font-size:12px;line-height:1.5;direction:ltr;text-align:left}
  .row2{display:grid;grid-template-columns:1fr 1fr;gap:8px}
  .chk{display:flex;align-items:center;gap:8px;margin:9px 11px;cursor:pointer;font-size:12.5px}
  .chk input{width:15px;height:15px;accent-color:var(--accent);cursor:pointer}
  .section-title{
    margin:14px 11px 4px;font-size:10.5px;font-weight:800;color:var(--accent);
    letter-spacing:.8px;text-transform:uppercase;border-bottom:1px solid var(--border);padding-bottom:5px;
  }
  .empty{text-align:center;color:var(--muted);padding:40px 20px;font-size:12px;line-height:2}
  .color-row{display:flex;gap:8px;align-items:center}
  .color-row input[type=color]{
    width:42px;height:32px;border:1px solid var(--border);border-radius:6px;
    background:#1a1a20;cursor:pointer;padding:2px;
  }
  .color-row input[type=text]{flex:1}
  /* ===== Assets ===== */
  #assets{border-top:1px solid var(--border);height:150px;display:flex;flex-direction:column;flex-shrink:0}
  #asset-grid{flex:1;overflow-y:auto;display:flex;flex-wrap:wrap;gap:7px;padding:9px;align-content:flex-start}
  .asset{
    width:64px;height:64px;background:#1a1a20;border:1px solid var(--border);border-radius:7px;
    display:flex;align-items:center;justify-content:center;overflow:hidden;cursor:pointer;position:relative;
  }
  .asset:hover{border-color:var(--accent)}
  .asset img{max-width:100%;max-height:100%;object-fit:contain;pointer-events:none}
  /* ===== Status bar ===== */
  #status{
    height:26px;background:#1f1f25;border-top:1px solid var(--border);
    display:flex;align-items:center;gap:18px;padding:0 12px;font-size:11px;color:var(--muted);
    flex-shrink:0;
  }
  #status b{color:var(--text);font-weight:600}
  .dot{width:7px;height:7px;border-radius:50%;display:inline-block;margin-left:5px}
  .dot.on{background:var(--green);box-shadow:0 0 8px var(--green)}
  .dot.off{background:#555}
  /* ===== Modal ===== */
  #modal{position:fixed;inset:0;background:rgba(0,0,0,.7);display:none;align-items:center;
    justify-content:center;z-index:100;backdrop-filter:blur(3px)}
  #modal.show{display:flex}
  .modal-box{background:var(--panel);border:1px solid var(--border);border-radius:12px;
    padding:22px;width:420px;max-width:90vw;box-shadow:0 20px 60px rgba(0,0,0,.6)}
  .modal-box h3{margin-bottom:14px;font-size:15px}
  .modal-box p{color:var(--muted);font-size:12.5px;line-height:1.8;margin-bottom:14px}
  .modal-box .btn{width:100%;justify-content:center;margin-top:8px}
</style>
</head>
<body>

<div id="toolbar">
  <div class="logo">⚡ GameForge</div>
  <div class="sep"></div>
  <button class="btn play" id="btnPlay">▶ تشغيل</button>
  <button class="btn pause" id="btnPause">⏸ إيقاف مؤقت</button>
  <button class="btn stop" id="btnStop">⏹ إيقاف</button>
  <div class="sep"></div>
  <button class="btn" id="btnSave">💾 حفظ</button>
  <button class="btn" id="btnLoad">📂 فتح</button>
  <button class="btn" id="btnExport">📦 تصدير لعبة</button>
  <div class="spacer"></div>
  <button class="btn" id="btnGrid">▦ الشبكة</button>
  <button class="btn" id="btnReset">↺ إعادة ضبط الكاميرا</button>
  <input type="file" id="fileInput" accept=".json" hidden>
  <input type="file" id="imgInput" accept="image/*" multiple hidden>
</div>

<div id="main">
  <!-- Hierarchy -->
  <div class="panel" id="left">
    <div class="panel-head">
      <span>المشهد</span>
      <span style="display:flex;gap:5px">
        <select id="addType" style="background:#2c2c34;border:1px solid #38383f;color:#e4e4e8;border-radius:5px;font-size:11px;padding:2px 4px;font-family:inherit">
          <option value="rect">▭ مربع</option>
          <option value="circle">◯ دائرة</option>
          <option value="text">T نص</option>
          <option value="sprite">🖼 صورة</option>
        </select>
        <button class="btn" id="btnAdd" style="padding:2px 8px;font-size:14px;line-height:1">+</button>
      </span>
    </div>
    <div class="panel-body" id="hierarchy"></div>
    <div id="assets">
      <div class="panel-head">
        <span>الأصول</span>
        <button class="btn" id="btnImport" style="padding:2px 8px;font-size:11px">⬆ استيراد</button>
      </div>
      <div id="asset-grid"></div>
    </div>
  </div>

  <!-- Viewport -->
  <div id="center">
    <div id="viewport-wrap">
      <canvas id="viewport"></canvas>
    </div>
  </div>

  <!-- Inspector -->
  <div class="panel" id="right">
    <div class="panel-head"><span>الخصائص</span></div>
    <div class="panel-body" id="inspector"></div>
  </div>
</div>

<div id="status">
  <span><span class="dot off" id="stateDot"></span><b id="stateTxt">المحرر</b></span>
  <span>الكائنات: <b id="statCount">0</b></span>
  <span>FPS: <b id="statFps">0</b></span>
  <span>التكبير: <b id="statZoom">100%</b></span>
  <span style="margin-right:auto;color:#666">اختصارات: حذف = Delete · تشغيل = مسافة · حفظ = Ctrl+S</span>
</div>

<div id="modal"><div class="modal-box" id="modalBox"></div></div>

<script>
/* ============================================================
   GameForge — 2D Browser Game Engine
   ============================================================ */

/* ---------- Utils ---------- */
const uid = () => 'o' + Math.random().toString(36).slice(2, 10);
const clamp = (v,a,b) => Math.min(b, Math.max(a, v));
const $ = id => document.getElementById(id);

/* ---------- State ---------- */
let scene = { name: 'مشروع جديد', objects: [] };
let selectedId = null;
let cam = { x: 0, y: 0, zoom: 1 };
let showGrid = true;
let running = false;      // play mode active
let paused = false;
let snapshot = null;      // scene backup for play mode
let assets = [];          // {name, src}
const imgCache = new Map();
const keys = Object.create(null);
let VW = 800, VH = 600;
let fps = 0, frames = 0, fpsTimer = 0;
let lastTime = performance.now();

/* ---------- Canvas ---------- */
const canvas = $('viewport');
const ctx = canvas.getContext('2d');

function resizeCanvas() {
  const wrap = canvas.parentElement;
  const r = wrap.getBoundingClientRect();
  const dpr = window.devicePixelRatio || 1;
  VW = r.width; VH = r.height;
  canvas.width = Math.max(1, r.width * dpr);
  canvas.height = Math.max(1, r.height * dpr);
  canvas.style.width = r.width + 'px';
  canvas.style.height = r.height + 'px';
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
}
window.addEventListener('resize', resizeCanvas);

/* ---------- Coordinates ---------- */
function w2s(wx, wy) {
  return { x: (wx - cam.x) * cam.zoom + VW / 2, y: (wy - cam.y) * cam.zoom + VH / 2 };
}
function s2w(sx, sy) {
  return { x: (sx - VW / 2) / cam.zoom + cam.x, y: (sy - VH / 2) / cam.zoom + cam.y };
}

/* ---------- Images ---------- */
function getImage(src) {
  if (!src) return null;
  if (!imgCache.has(src)) {
    const im = new Image();
    im.src = src;
    imgCache.set(src, im);
  }
  return imgCache.get(src);
}

/* ---------- GameObject ---------- */
function makeObject(type) {
  const o = {
    id: uid(),
    name: ({ rect: 'مربع', circle: 'دائرة', text: 'نص', sprite: 'صورة' })[type] || 'كائن',
    type,
    x: 0, y: 0,
    w: 100, h: 100,
    rotation: 0,
    scaleX: 1, scaleY: 1,
    color: '#4a9eff',
    visible: true,
    sprite: null,
    text: 'مرحباً',
    fontSize: 32,
    script: '',
    physics: { enabled: false, static: false, gravity: 0, vx: 0, vy: 0 },
    _f: null
  };
  if (type === 'circle') { o.w = 100; o.h = 100; }
  if (type === 'text') { o.w = 240; o.h = 60; o.color = '#ffffff'; }
  if (type === 'sprite') { o.w = 128; o.h = 128; }
  return o;
}

const TYPE_ICON = { rect: '▭', circle: '◯', text: 'T', sprite: '🖼' };

function getSelected() {
  return scene.objects.find(o => o.id === selectedId) || null;
}

/* ---------- Rendering ---------- */
function drawGrid() {
  if (!showGrid) return;
  const step = 50 * cam.zoom;
  if (step < 7) return;
  const originS = w2s(0, 0);
  let ox = originS.x % step; if (ox < 0) ox += step;
  let oy = originS.y % step; if (oy < 0) oy += step;

  ctx.strokeStyle = '#232329';
  ctx.lineWidth = 1;
  ctx.beginPath();
  for (let x = ox; x < VW; x += step) { ctx.moveTo(Math.round(x) + .5, 0); ctx.lineTo(Math.round(x) + .5, VH); }
  for (let y = oy; y < VH; y += step) { ctx.moveTo(0, Math.round(y) + .5); ctx.lineTo(VW, Math.round(y) + .5); }
  ctx.stroke();

  // axes
  ctx.strokeStyle = '#3a2f4a';
  ctx.beginPath();
  ctx.moveTo(Math.round(originS.x) + .5, 0); ctx.lineTo(Math.round(originS.x) + .5, VH);
  ctx.moveTo(0, Math.round(originS.y) + .5); ctx.lineTo(VW, Math.round(originS.y) + .5);
  ctx.stroke();

  // axis colors
  ctx.strokeStyle = 'rgba(255,90,90,.35)';
  ctx.beginPath(); ctx.moveTo(originS.x, originS.y); ctx.lineTo(originS.x + 40, originS.y); ctx.stroke();
  ctx.strokeStyle = 'rgba(90,255,140,.35)';
  ctx.beginPath(); ctx.moveTo(originS.x, originS.y); ctx.lineTo(originS.x, originS.y - 40); ctx.stroke();
}

function drawObject(o) {
  if (!o.visible) return;
  const s = w2s(o.x, o.y);
  const z = cam.zoom;

  ctx.save();
  ctx.translate(s.x, s.y);
  ctx.rotate(o.rotation * Math.PI / 180);
  ctx.scale(o.scaleX, o.scaleY);

  const w = o.w * z, h = o.h * z;

  if (o.type === 'rect') {
    ctx.fillStyle = o.color;
    ctx.fillRect(-w / 2, -h / 2, w, h);
    ctx.strokeStyle = 'rgba(255,255,255,.12)';
    ctx.lineWidth = 1;
    ctx.strokeRect(-w / 2, -h / 2, w, h);
  } else if (o.type === 'circle') {
    ctx.fillStyle = o.color;
    ctx.beginPath();
    ctx.ellipse(0, 0, Math.abs(w / 2), Math.abs(h / 2), 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.strokeStyle = 'rgba(255,255,255,.12)';
    ctx.lineWidth = 1;
    ctx.stroke();
  } else if (o.type === 'sprite') {
    const im = getImage(o.sprite);
    if (im && im.complete && im.naturalWidth) {
      ctx.drawImage(im, -w / 2, -h / 2, w, h);
    } else {
      ctx.fillStyle = '#333';
      ctx.fillRect(-w / 2, -h / 2, w, h);
      ctx.fillStyle = '#555';
      ctx.font = (14 * z) + 'px sans-serif';
      ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
      ctx.fillText('🖼', 0, 0);
    }
  } else if (o.type === 'text') {
    ctx.fillStyle = o.color;
    ctx.font = '700 ' + (o.fontSize * z) + 'px "Segoe UI",Tahoma,sans-serif';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(o.text, 0, 0);
  }
  ctx.restore();
}

function getCorners(o) {
  const hw = o.w * o.scaleX / 2, hh = o.h * o.scaleY / 2;
  const rad = o.rotation * Math.PI / 180;
  const c = Math.cos(rad), s = Math.sin(rad);
  return [[-hw, -hh], [hw, -hh], [hw, hh], [-hw, hh]].map(([x, y]) => ({
    x: o.x + x * c - y * s,
    y: o.y + x * s + y * c
  }));
}

function drawSelection(o) {
  const corners = getCorners(o).map(p => w2s(p.x, p.y));
  ctx.save();
  ctx.strokeStyle = '#4a9eff';
  ctx.lineWidth = 1.5;
  ctx.setLineDash([]);
  ctx.beginPath();
  ctx.moveTo(corners[0].x, corners[0].y);
  for (let i = 1; i < 4; i++) ctx.lineTo(corners[i].x, corners[i].y);
  ctx.closePath();
  ctx.stroke();

  // handles
  ctx.fillStyle = '#4a9eff';
  corners.forEach(p => {
    ctx.fillRect(p.x - 3.5, p.y - 3.5, 7, 7);
  });

  // rotation handle line
  const topMid = { x: (corners[0].x + corners[1].x) / 2, y: (corners[0].y + corners[1].y) / 2 };
  ctx.beginPath();
  ctx.moveTo(topMid.x, topMid.y);
  ctx.lineTo(topMid.x, topMid.y - 26);
  ctx.stroke();
  ctx.beginPath();
  ctx.arc(topMid.x, topMid.y - 32, 5, 0, Math.PI * 2);
  ctx.fill();
  ctx.restore();
}

/* ---------- Physics ---------- */
function aabb(o) {
  const hw = Math.abs(o.w * o.scaleX) / 2;
  const hh = Math.abs(o.h * o.scaleY) / 2;
  return { l: o.x - hw, r: o.x + hw, t: o.y - hh, b: o.y + hh };
}

function resolveCollision(a, b) {
  const A = aabb(a), B = aabb(b);
  if (A.r <= B.l || A.l >= B.r || A.b <= B.t || A.t >= B.b) return;

  const aStatic = a.physics.static, bStatic = b.physics.static;
  if (aStatic && bStatic) return;

  const ox = Math.min(A.r - B.l, B.r - A.l);
  const oy = Math.min(A.b - B.t, B.b - A.t);

  if (ox < oy) {
    const dir = (A.l + A.r) / 2 < (B.l + B.r) / 2 ? -1 : 1;
    if (!aStatic && !bStatic) { a.x += dir * ox / 2; b.x -= dir * ox / 2; }
    else if (!aStatic) a.x += dir * ox;
    else b.x -= dir * ox;
    if (!aStatic) a.physics.vx = 0;
    if (!bStatic) b.physics.vx = 0;
  } else {
    const dir = (A.t + A.b) / 2 < (B.t + B.b) / 2 ? -1 : 1;
    if (!aStatic && !bStatic) { a.y += dir * oy / 2; b.y -= dir * oy / 2; }
    else if (!aStatic) a.y += dir * oy;
    else b.y -= dir * oy;
    if (!aStatic) a.physics.vy = 0;
    if (!bStatic) b.physics.vy = 0;
  }
}

/* ---------- Scripts ---------- */
function compileScripts() {
  scene.objects.forEach(o => {
    o._f = null;
    if (o.script && o.script.trim()) {
      try {
        o._f = new Function('obj', 'scene', 'dt', 'input', 'api',
          '"use strict";\n' + o.script);
      } catch (e) {
        console.error('خطأ في سكربت "' + o.name + '":', e.message);
      }
    }
  });
}

const scriptAPI = {
  find: name => scene.objects.find(o => o.name === name),
  log: (...a) => console.log('[GameForge]', ...a)
};

function runScripts(dt) {
  for (const o of scene.objects) {
    if (!o._f) continue;
    try {
      o._f(o, scene, dt, keys, scriptAPI);
    } catch (e) {
      console.error('خطأ وقت التشغيل في "' + o.name + '":', e.message);
      o._f = null;
    }
  }
}

function physicsStep(dt) {
  for (const o of scene.objects) {
    if (!o.physics || !o.physics.enabled || o.physics.static) continue;
    o.physics.vy += (o.physics.gravity || 0) * dt;
    o.x += o.physics.vx * dt;
    o.y += o.physics.vy * dt;
  }
  const objs = scene.objects;
  for (let i = 0; i < objs.length; i++) {
    for (let j = i + 1; j < objs.length; j++) {
      const a = objs[i], b = objs[j];
      if (!a.physics || !a.physics.enabled) continue;
      if (!b.physics || !b.physics.enabled) continue;
      resolveCollision(a, b);
    }
  }
}

/* ---------- Main Loop ---------- */
function frame(now) {
  let dt = (now - lastTime) / 1000;
  lastTime = now;
  dt = Math.min(dt, 0.05);

  // FPS
  frames++;
  fpsTimer += dt;
  if (fpsTimer >= 0.5) {
    fps = Math.round(frames / fpsTimer);
    frames = 0;
    fpsTimer = 0;
    $('statFps').textContent = fps;
  }

  // Update
  if (running && !paused) {
    runScripts(dt);
    physicsStep(dt);
  }

  // Render
  ctx.fillStyle = '#151519';
  ctx.fillRect(0, 0, VW, VH);
  drawGrid();

  for (const o of scene.objects) drawObject(o);

  if (!running) {
    const sel = getSelected();
    if (sel) drawSelection(sel);
  }

  requestAnimationFrame(frame);
}

/* ---------- Hierarchy UI ---------- */
function renderHierarchy() {
  const el = $('hierarchy');
  el.innerHTML = '';
  if (!scene.objects.length) {
    el.innerHTML = '<div class="empty">لا توجد كائنات<br><span style="font-size:11px">اضغط + لإضافة كائن</span></div>';
    return;
  }
  scene.objects.forEach(o => {
    const d = document.createElement('div');
    d.className = 'hi' + (o.id === selectedId ? ' sel' : '');
    d.innerHTML = '<span class="ic">' + (TYPE_ICON[o.type] || '▭') + '</span>' +
                  '<span class="nm">' + escapeHtml(o.name) + '</span>' +
                  '<span class="del">✕</span>';
    d.addEventListener('click', e => {
      if (e.target.classList.contains('del')) {
        deleteObject(o.id);
      } else {
        selectObject(o.id);
      }
    });
    el.appendChild(d);
  });
  $('statCount').textContent = scene.objects.length;
}

function escapeHtml(s) {
  return String(s).replace(/[&<>"']/g, c => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]));
}

function selectObject(id) {
  selectedId = id;
  renderHierarchy();
  buildInspector();
}

function deleteObject(id) {
  scene.objects = scene.objects.filter(o => o.id !== id);
  if (selectedId === id) selectedId = null;
  renderHierarchy();
  buildInspector();
}

/* ---------- Inspector UI ---------- */
const inp = {};

function mkField(label, inputEl) {
  const d = document.createElement('div');
  d.className = 'field';
  const l = document.createElement('label');
  l.textContent = label;
  d.appendChild(l);
  d.appendChild(inputEl);
  return d;
}

function numInput(value, onChange, step) {
  const i = document.createElement('input');
  i.type = 'number';
  i.step = step || 1;
  i.value = Math.round(value * 100) / 100;
  i.addEventListener('input', () => onChange(parseFloat(i.value) || 0));
  return i;
}

function buildInspector() {
  const el = $('inspector');
  el.innerHTML = '';
  Object.keys(inp).forEach(k => delete inp[k]);

  const o = getSelected();
  if (!o) {
    el.innerHTML = '<div class="empty">اختر كائناً من المشهد<br>لعرض وتعديل خصائصه</div>';
    return;
  }

  // --- Identity ---
  const nameI = document.createElement('input');
  nameI.type = 'text';
  nameI.value = o.name;
  nameI.addEventListener('input', () => { o.name = nameI.value; renderHierarchy(); });
  el.appendChild(mkField('اسم الكائن', nameI));

  const sec1 = document.createElement('div');
  sec1.className = 'section-title';
  sec1.textContent = 'التحويل';
  el.appendChild(sec1);

  // Position
  const posDiv = document.createElement('div');
  posDiv.className = 'field';
  const posL = document.createElement('label');
  posL.textContent = 'الموقع (X / Y)';
  const posRow = document.createElement('div');
  posRow.className = 'row2';
  inp.x = numInput(o.x, v => { o.x = v; });
  inp.y = numInput(o.y, v => { o.y = v; });
  posRow.appendChild(inp.x);
  posRow.appendChild(inp.y);
  posDiv.appendChild(posL);
  posDiv.appendChild(posRow);
  el.appendChild(posDiv);

  // Size
  const sizeDiv = document.createElement('div');
  sizeDiv.className = 'field';
  const sizeL = document.createElement('label');
  sizeL.textContent = 'الحجم (العرض / الارتفاع)';
  const sizeRow = document.createElement('div');
  sizeRow.className = 'row2';
  inp.w = numInput(o.w, v => { o.w = Math.max(1, v); });
  inp.h = numInput(o.h, v => { o.h = Math.max(1, v); });
  sizeRow.appendChild(inp.w);
  sizeRow.appendChild(inp.h);
  sizeDiv.appendChild(sizeL);
  sizeDiv.appendChild(sizeRow);
  el.appendChild(sizeDiv);

  // Rotation / Scale
  const rsDiv = document.createElement('div');
  rsDiv.className = 'field';
  const rsL = document.createElement('label');
  rsL.textContent = 'الدوران (درجة) / التكبير';
  const rsRow = document.createElement('div');
  rsRow.className = 'row2';
  inp.rot = numInput(o.rotation, v => { o.rotation = v; });
  inp.scl = numInput(o.scaleX, v => { o.scaleX = v; o.scaleY = v; });
  rsRow.appendChild(inp.rot);
  rsRow.appendChild(inp.scl);
  rsDiv.appendChild(rsL);
  rsDiv.appendChild(rsRow);
  el.appendChild(rsDiv);

  // Visible
  const visLbl = document.createElement('label');
  visLbl.className = 'chk';
  const visChk = document.createElement('input');
  visChk.type = 'checkbox';
  visChk.checked = o.visible;
  visChk.addEventListener('change', () => { o.visible = visChk.checked; });
  visLbl.appendChild(visChk);
  visLbl.appendChild(document.createTextNode('مرئي في المشهد'));
  el.appendChild(visLbl);

  // --- Appearance ---
  const sec2 = document.createElement('div');
  sec2.className = 'section-title';
  sec2.textContent = 'المظهر';
  el.appendChild(sec2);

  if (o.type === 'text') {
    const tI = document.createElement('input');
    tI.type = 'text';
    tI.value = o.text;
    tI.addEventListener('input', () => { o.text = tI.value; });
    el.appendChild(mkField('النص', tI));

    el.appendChild(mkField('حجم الخط', numInput(o.fontSize, v => { o.fontSize = Math.max(4, v); })));
  }

  if (o.type === 'sprite') {
    const btn = document.createElement('button');
    btn.className = 'btn';
    btn.style.width = '100%';
    btn.textContent = '🖼 اختيار صورة من الأصول';
    btn.addEventListener('click', () => {
      if (!assets.length) { alert('لا توجد أصول. استورد صورة أولاً من لوحة الأصول.'); return; }
      const a = assets[0];
      o.sprite = a.src;
      o.w = 128; o.h = 128;
      buildInspector();
    });
    el.appendChild(mkField('الصورة', btn));
  }

  // Color
  const colDiv = document.createElement('div');
  colDiv.className = 'field';
  const colL = document.createElement('label');
  colL.textContent = 'اللون';
  const colRow = document.createElement('div');
  colRow.className = 'color-row';
  const colorPick = document.createElement('input');
  colorPick.type = 'color';
  colorPick.value = o.color;
  const colorTxt = document.createElement('input');
  colorTxt.type = 'text';
  colorTxt.value = o.color;
  colorPick.addEventListener('input', () => { o.color = colorPick.value; colorTxt.value = colorPick.value; });
  colorTxt.addEventListener('input', () => { o.color = colorTxt.value; colorPick.value = colorTxt.value; });
  colRow.appendChild(colorPick);
  colRow.appendChild(colorTxt);
  colDiv.appendChild(colL);
  colDiv.appendChild(colRow);
  el.appendChild(colDiv);

  // --- Physics ---
  const sec3 = document.createElement('div');
  sec3.className = 'section-title';
  sec3.textContent = 'الفيزياء';
  el.appendChild(sec3);

  const phLbl = document.createElement('label');
  phLbl.className = 'chk';
  const phChk = document.createElement('input');
  phChk.type = 'checkbox';
  phChk.checked = o.physics.enabled;
  phChk.addEventListener('change', () => {
    o.physics.enabled = phChk.checked;
    if (phChk.checked && !o.physics.gravity) o.physics.gravity = 1500;
    buildInspector();
  });
  phLbl.appendChild(phChk);
  phLbl.appendChild(document.createTextNode('تفعيل الفيزياء'));
  el.appendChild(phLbl);

  if (o.physics.enabled) {
    const stLbl = document.createElement('label');
    stLbl.className = 'chk';
    const stChk = document.createElement('input');
    stChk.type = 'checkbox';
    stChk.checked = o.physics.static;
    stChk.addEventListener('change', () => { o.physics.static = stChk.checked; });
    stLbl.appendChild(stChk);
    stLbl.appendChild(document.createTextNode('جسم ثابت (لا يتأثر بالجاذبية)'));
    el.appendChild(stLbl);

    if (!o.physics.static) {
      el.appendChild(mkField('الجاذبية', numInput(o.physics.gravity, v => { o.physics.gravity = v; }, 10)));
      el.appendChild(mkField('السرعة الأفقية (vx)', numInput(o.physics.vx, v => { o.physics.vx = v; }, 10)));
      el.appendChild(mkField('السرعة الرأسية (vy)', numInput(o.physics.vy, v => { o.physics.vy = v; }, 10)));
    }
  }

  // --- Script ---
  const sec4 = document.createElement('div');
  sec4.className = 'section-title';
  sec4.textContent = 'السكربت (JavaScript)';
  el.appendChild(sec4);

  const hint = document.createElement('div');
  hint.style.cssText = 'margin:0 11px 6px;font-size:10.5px;color:#7a7a88;line-height:1.7';
  hint.innerHTML = 'المتغيرات المتاحة: <b>obj</b> · <b>scene</b> · <b>dt</b> · <b>input</b> · <b>api</b>';
  el.appendChild(hint);

  const ta = document.createElement('textarea');
  ta.value = o.script;
  ta.spellcheck = false;
  ta.addEventListener('input', () => { o.script = ta.value; });
  const taWrap = document.createElement('div');
  taWrap.className = 'field';
  taWrap.appendChild(ta);
  el.appendChild(taWrap);

  // Actions
  const sec5 = document.createElement('div');
  sec5.className = 'section-title';
  sec5.textContent = 'إجراءات';
  el.appendChild(sec5);

  const dupBtn = document.createElement('button');
  dupBtn.className = 'btn';
  dupBtn.style.width = 'calc(100% - 22px)';
  dupBtn.style.margin = '0 11px 8px';
  dupBtn.textContent = '⧉ نسخ الكائن';
  dupBtn.addEventListener('click', () => {
    const copy = JSON.parse(JSON.stringify(o));
    copy.id = uid();
    copy.name = o.name + ' (نسخة)';
    copy.x += 40;
    copy.y += 40;
    scene.objects.push(copy);
    selectObject(copy.id);
  });
  el.appendChild(dupBtn);

  const delBtn = document.createElement('button');
  delBtn.className = 'btn stop';
  delBtn.style.width = 'calc(100% - 22px)';
  delBtn.style.margin = '0 11px 20px';
  delBtn.textContent = '🗑 حذف الكائن';
  delBtn.addEventListener('click', () => deleteObject(o.id));
  el.appendChild(delBtn);
}

/* ---------- Assets ---------- */
function renderAssets() {
  const g = $('asset-grid');
  g.innerHTML = '';
  if (!assets.length) {
    g.innerHTML = '<div style="color:#666;font-size:11px;padding:8px">لا توجد أصول مستوردة</div>';
    return;
  }
  assets.forEach(a => {
    const d = document.createElement('div');
    d.className = 'asset';
    d.title = a.name;
    const img = document.createElement('img');
    img.src = a.src;
    d.appendChild(img);
    d.addEventListener('click', () => {
      const o = makeObject('sprite');
      o.sprite = a.src;
      o.name = a.name;
      o.x = cam.x;
      o.y = cam.y;
      scene.objects.push(o);
      selectObject(o.id);
    });
    g.appendChild(d);
  });
}

/* ---------- Mouse Interaction ---------- */
let dragMode = null;      // 'move' | 'resize' | 'rotate' | 'pan'
let dragStart = null;     // {sx, sy, wx, wy}
let dragObjStart = null;
let panStart = null;

function mousePos(e) {
  const r = canvas.getBoundingClientRect();
  return { x: e.clientX - r.left, y: e.clientY - r.top };
}

function hitTest(o, wx, wy) {
  const rad = -o.rotation * Math.PI / 180;
  const dx = wx - o.x, dy = wy - o.y;
  const lx = dx * Math.cos(rad) - dy * Math.sin(rad);
  const ly = dx * Math.sin(rad) + dy * Math.cos(rad);
  const hw = Math.abs(o.w * o.scaleX) / 2;
  const hh = Math.abs(o.h * o.scaleY) / 2;
  if (o.type === 'circle') {
    if (hw === 0 || hh === 0) return false;
    return (lx * lx) / (hw * hw) + (ly * ly) / (hh * hh) <= 1;
  }
  return Math.abs(lx) <= hw && Math.abs(ly) <= hh;
}

function pickObject(wx, wy) {
  for (let i = scene.objects.length - 1; i >= 0; i--) {
    const o = scene.objects[i];
    if (o.visible && hitTest(o, wx, wy)) return o;
  }
  return null;
}

function nearHandle(o, sx, sy) {
  const corners = getCorners(o).map(p => w2s(p.x, p.y));
  // resize handle = bottom-right corner (index 2)
  const br = corners[2];
  if (Math.hypot(br.x - sx, br.y - sy) < 9) return 'resize';
  // rotate handle
  const topMid = { x: (corners[0].x + corners[1].x) / 2, y: (corners[0].y + corners[1].y) / 2 };
  const rotH = { x: topMid.x, y: topMid.y - 32 };
  if (Math.hypot(rotH.x - sx, rotH.y - sy) < 9) return 'rotate';
  return null;
}

canvas.addEventListener('mousedown', e => {
  if (e.button !== 0) return;
  const p = mousePos(e);
  const w = s2w(p.x, p.y);
  dragStart = { sx: p.x, sy: p.y, wx: w.x, wy: w.y };

  if (running) {
    dragMode = 'pan';
    panStart = { x: cam.x, y: cam.y };
    canvas.classList.add('panning');
    return;
  }

  const sel = getSelected();
  if (sel) {
    const h = nearHandle(sel, p.x, p.y);
    if (h) {
      dragMode = h;
      dragObjStart = { x: sel.x, y: sel.y, w: sel.w, h: sel.h, rot: sel.rotation };
      canvas.classList.add('moving');
      return;
    }
  }

  const hit = pickObject(w.x, w.y);
  if (hit) {
    selectObject(hit.id);
    dragMode = 'move';
    dragObjStart = { x: hit.x, y: hit.y };
    canvas.classList.add('moving');
  } else {
    selectObject(null);
    dragMode = 'pan';
    panStart = { x: cam.x, y: cam.y };
    canvas.classList.add('panning');
  }
});

canvas.addEventListener('mousemove', e => {
  const p = mousePos(e);
  if (!dragMode) {
    const w = s2w(p.x, p.y);
    const hit = pickObject(w.x, w.y);
    canvas.style.cursor = hit ? 'grab' : 'default';
    return;
  }
  const o = getSelected();

  if (dragMode === 'pan') {
    cam.x = panStart.x - (p.x - dragStart.sx) / cam.zoom;
    cam.y = panStart.y - (p.y - dragStart.sy) / cam.zoom;
    return;
  }
  if (!o) return;

  const dxw = (p.x - dragStart.sx) / cam.zoom;
  const dyw = (p.y - dragStart.sy) / cam.zoom;

  if (dragMode === 'move') {
    o.x = Math.round((dragObjStart.x + dxw) * 10) / 10;
    o.y = Math.round((dragObjStart.y + dyw) * 10) / 10;
    if (inp.x) inp.x.value = o.x;
    if (inp.y) inp.y.value = o.y;
  } else if (dragMode === 'resize') {
    o.w = Math.max(5, Math.round((dragObjStart.w + dxw * 2) * 10) / 10);
    o.h = Math.max(5, Math.round((dragObjStart.h + dyw * 2) * 10) / 10);
    if (inp.w) inp.w.value = o.w;
    if (inp.h) inp.h.value = o.h;
  } else if (dragMode === 'rotate') {
    const ang = Math.atan2(p.y - (VH / 2 + (o.y - cam.y) * cam.zoom) * 0 - 0, 0);
    // simpler: compute angle from object center in screen space
    const cs = w2s(o.x, o.y);
    const a = Math.atan2(p.y - cs.y, p.x - cs.x) * 180 / Math.PI + 90;
    o.rotation = Math.round(a * 10) / 10;
    if (inp.rot) inp.rot.value = o.rotation;
  }
});

function endDrag() {
  dragMode = null;
  dragObjStart = null;
  canvas.classList.remove('moving', 'panning');
  if (!running) buildInspector();
}
canvas.addEventListener('mouseup', endDrag);
canvas.addEventListener('mouseleave', endDrag);

canvas.addEventListener('wheel', e => {
  e.preventDefault();
  const p = mousePos(e);
  const before = s2w(p.x, p.y);
  const factor = e.deltaY < 0 ? 1.12 : 1 / 1.12;
  cam.zoom = clamp(cam.zoom * factor, 0.08, 8);
  const after = s2w(p.x, p.y);
  cam.x += before.x - after.x;
  cam.y += before.y - after.y;
  $('statZoom').textContent = Math.round(cam.zoom * 100) + '%';
}, { passive: false });

/* ---------- Keyboard ---------- */
window.addEventListener('keydown', e => {
  const tag = (e.target.tagName || '').toUpperCase();
  if (tag === 'INPUT' || tag === 'TEXTAREA' || tag === 'SELECT') return;

  keys[e.key.toLowerCase()] = true;

  if (e.key === 'Delete' && selectedId && !running) {
    deleteObject(selectedId);
    e.preventDefault();
  }
  if (e.key === ' ' && !running) {
    e.preventDefault();
    startPlay();
  }
  if ((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 's') {
    e.preventDefault();
    saveProject();
  }
  if (e.key === 'Escape' && running) {
    stopPlay();
  }
});
window.addEventListener('keyup', e => {
  keys[e.key.toLowerCase()] = false;
});
window.addEventListener('blur', () => {
  for (const k in keys) keys[k] = false;
});

/* ---------- Play Mode ---------- */
function startPlay() {
  if (running) return;
  snapshot = JSON.parse(JSON.stringify({ objects: scene.objects }));
  // reset velocities
  scene.objects.forEach(o => {
    if (o.physics) { o.physics.vx = 0; o.physics.vy = 0; }
  });
  compileScripts();
  running = true;
  paused = false;
  selectedId = null;
  renderHierarchy();
  buildInspector();
  $('btnPlay').classList.add('on');
  $('stateDot').className = 'dot on';
  $('stateTxt').textContent = 'قيد التشغيل';
  canvas.style.cursor = 'default';
}

function pausePlay() {
  if (!running) return;
  paused = !paused;
  $('stateTxt').textContent = paused ? 'متوقف مؤقتاً' : 'قيد التشغيل';
  $('stateDot').className = paused ? 'dot off' : 'dot on';
}

function stopPlay() {
  if (!running) return;
  scene.objects = snapshot.objects;
  snapshot = null;
  running = false;
  paused = false;
  scene.objects.forEach(o => { o._f = null; });
  renderHierarchy();
  buildInspector();
  $('btnPlay').classList.remove('on');
  $('stateDot').className = 'dot off';
  $('stateTxt').textContent = 'المحرر';
}

/* ---------- Save / Load ---------- */
function saveProject() {
  const data = {
    format: 'gameforge-project',
    version: 1,
    name: scene.name,
    camera: { x: cam.x, y: cam.y, zoom: cam.zoom },
    objects: scene.objects.map(o => ({
      id: o.id, name: o.name, type: o.type,
      x: o.x, y: o.y, w: o.w, h: o.h,
      rotation: o.rotation, scaleX: o.scaleX, scaleY: o.scaleY,
      color: o.color, visible: o.visible,
      sprite: o.sprite, text: o.text, fontSize: o.fontSize,
      script: o.script, physics: o.physics
    })),
    assets
  };
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = (scene.name || 'project') + '.gfproj.json';
  a.click();
  URL.revokeObjectURL(a.href);
}

function loadProject(file) {
  const reader = new FileReader();
  reader.onload = () => {
    try {
      const d = JSON.parse(reader.result);
      if (!d.objects) throw new Error('ملف غير صالح');
      scene = {
        name: d.name || 'مشروع',
        objects: d.objects.map(o => Object.assign(makeObject(o.type || 'rect'), o))
      };
      scene.objects.forEach(o => {
        if (!o.physics) o.physics = { enabled: false, static: false, gravity: 0, vx: 0, vy: 0 };
        o._f = null;
      });
      assets = d.assets || [];
      if (d.camera) { cam = { ...d.camera }; }
      selectedId = null;
      renderHierarchy();
      renderAssets();
      buildInspector();
      $('statZoom').textContent = Math.round(cam.zoom * 100) + '%';
    } catch (err) {
      alert('تعذّر فتح الملف: ' + err.message);
    }
  };
  reader.readAsText(file);
}

/* ---------- Export Standalone Game ---------- */
function exportGame() {
  const data = JSON.stringify({
    objects: scene.objects.map(o => ({
      x: o.x, y: o.y, w: o.w, h: o.h,
      rotation: o.rotation, scaleX: o.scaleX, scaleY: o.scaleY,
      type: o.type, color: o.color, visible: o.visible,
      sprite: o.sprite, text: o.text, fontSize: o.fontSize,
      script: o.script, physics: o.physics
    }))
  });

  const rt = [
    'var SCENE=' + data + ';',
    'var c=document.getElementById("c"),x=c.getContext("2d");',
    'var VW=0,VH=0;',
    'function rs(){var d=window.devicePixelRatio||1;VW=innerWidth;VH=innerHeight;c.width=VW*d;c.height=VH*d;c.style.width=VW+"px";c.style.height=VH+"px";x.setTransform(d,0,0,d,0,0);}',
    'addEventListener("resize",rs);rs();',
    'var keys={};',
    'addEventListener("keydown",function(e){keys[e.key.toLowerCase()]=1;});',
    'addEventListener("keyup",function(e){keys[e.key.toLowerCase()]=0;});',
    'var imgc={};',
    'function gi(s){if(!s)return null;if(!imgc[s]){var i=new Image();i.src=s;imgc[s]=i;}return imgc[s];}',
    'var O=SCENE.objects;',
    'O.forEach(function(o){if(o.script&&o.script.trim()){try{o._f=new Function("obj","scene","dt","input","api",o.script);}catch(e){console.error(e);}}});',
    'function aabb(o){var hw=Math.abs(o.w*o.scaleX)/2,hh=Math.abs(o.h*o.scaleY)/2;return{l:o.x-hw,r:o.x+hw,t:o.y-hh,b:o.y+hh};}',
    'function coll(a,b){var A=aabb(a),B=aabb(b);if(A.r<=B.l||A.l>=B.r||A.b<=B.t||A.t>=B.b)return;',
    'var as=a.physics.static,bs=b.physics.static;if(as&&bs)return;',
    'var ox=Math.min(A.r-B.l,B.r-A.l),oy=Math.min(A.b-B.t,B.b-A.t);',
    'if(ox<oy){var d=(A.l+A.r)/2<(B.l+B.r)/2?-1:1;',
    'if(!as&&!bs){a.x+=d*ox/2;b.x-=d*ox/2;}else if(!as){a.x+=d*ox;}else{b.x-=d*ox;}',
    'if(!as)a.physics.vx=0;if(!bs)b.physics.vx=0;}',
    'else{var d2=(A.t+A.b)/2<(B.t+B.b)/2?-1:1;',
    'if(!as&&!bs){a.y+=d2*oy/2;b.y-=d2*oy/2;}else if(!as){a.y+=d2*oy;}else{b.y-=d2*oy;}',
    'if(!as)a.physics.vy=0;if(!bs)b.physics.vy=0;}}',
    'var API={find:function(n){return O.find(function(q){return q.name===n;});},log:function(){console.log.apply(console,arguments);}};',
    'function step(dt){',
    '  O.forEach(function(o){if(o._f){try{o._f(o,SCENE,dt,keys,API);}catch(e){console.error(e);}}});',
    '  O.forEach(function(o){if(o.physics&&o.physics.enabled&&!o.physics.static){o.physics.vy+=(o.physics.gravity||0)*dt;o.x+=o.physics.vx*dt;o.y+=o.physics.vy*dt;}});',
    '  for(var i=0;i<O.length;i++)for(var j=i+1;j<O.length;j++){var a=O[i],b=O[j];',
    '    if(!(a.physics&&a.physics.enabled)||!(b.physics&&b.physics.enabled))continue;coll(a,b);}}',
    'function draw(o){if(!o.visible)return;',
    '  var sx=o.x+VW/2,sy=o.y+VH/2;',
    '  x.save();x.translate(sx,sy);x.rotate(o.rotation*Math.PI/180);x.scale(o.scaleX,o.scaleY);',
    '  var w=o.w,h=o.h;',
    '  if(o.type==="rect"){x.fillStyle=o.color;x.fillRect(-w/2,-h/2,w,h);}',
    '  else if(o.type==="circle"){x.fillStyle=o.color;x.beginPath();x.ellipse(0,0,Math.abs(w/2),Math.abs(h/2),0,0,Math.PI*2);x.fill();}',
    '  else if(o.type==="sprite"){var im=gi(o.sprite);if(im&&im.complete&&im.naturalWidth){x.drawImage(im,-w/2,-h/2,w,h);}else{x.fillStyle="#333";x.fillRect(-w/2,-h/2,w,h);}}',
    '  else if(o.type==="text"){x.fillStyle=o.color;x.font="700 "+o.fontSize+"px sans-serif";x.textAlign="center";x.textBaseline="middle";x.fillText(o.text,0,0);}',
    '  x.restore();}',
    'var last=performance.now();',
    'function loop(t){var dt=Math.min((t-last)/1000,0.05);last=t;step(dt);',
    '  x.fillStyle="#151519";x.fillRect(0,0,VW,VH);',
    '  for(var i=0;i<O.length;i++)draw(O[i]);',
    '  requestAnimationFrame(loop);}',
    'requestAnimationFrame(loop);'
  ].join('\n');

  const html = '<!DOCTYPE html>\n<html lang="ar" dir="rtl">\n<head>\n<meta charset="UTF-8">\n' +
    '<meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">\n' +
    '<title>' + escapeHtml(scene.name || 'لعبتي') + '</title>\n' +
    '<style>html,body{margin:0;padding:0;overflow:hidden;background:#151519;height:100%}' +
    'canvas{display:block;touch-action:none}</style>\n</head>\n<body>\n' +
    '<canvas id="c"></canvas>\n<script>\n' + rt + '\n<\/script>\n</body>\n</html>';

  const blob = new Blob([html], { type: 'text/html' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = (scene.name || 'game') + '.html';
  a.click();
  URL.revokeObjectURL(a.href);
}

/* ---------- Modal ---------- */
function showModal(html) {
  $('modalBox').innerHTML = html;
  $('modal').classList.add('show');
}
$('modal').addEventListener('click', e => {
  if (e.target.id === 'modal') $('modal').classList.remove('show');
});

/* ---------- Toolbar Wiring ---------- */
$('btnPlay').addEventListener('click', startPlay);
$('btnPause').addEventListener('click', pausePlay);
$('btnStop').addEventListener('click', stopPlay);

$('btnSave').addEventListener('click', saveProject);
$('btnLoad').addEventListener('click', () => $('fileInput').click());
$('fileInput').addEventListener('change', e => {
  if (e.target.files[0]) { loadProject(e.target.files[0]); e.target.value = ''; }
});

$('btnExport').addEventListener('click', () => {
  if (!scene.objects.length) { alert('أضف كائنات أولاً قبل التصدير.'); return; }
  exportGame();
});

$('btnGrid').addEventListener('click', () => {
  showGrid = !showGrid;
  $('btnGrid').style.opacity = showGrid ? '1' : '.5';
});

$('btnReset').addEventListener('click', () => {
  cam = { x: 0, y: 0, zoom: 1 };
  $('statZoom').textContent = '100%';
});

$('btnAdd').addEventListener('click', () => {
  const type = $('addType').value;
  const o = makeObject(type);
  o.x = Math.round(cam.x);
  o.y = Math.round(cam.y);
  scene.objects.push(o);
  selectObject(o.id);
});

$('btnImport').addEventListener('click', () => $('imgInput').click());
$('imgInput').addEventListener('change', e => {
  const files = Array.from(e.target.files);
  let pending = files.length;
  if (!pending) return;
  files.forEach(f => {
    const r = new FileReader();
    r.onload = () => {
      assets.push({ name: f.name.replace(/\.[^.]+$/, ''), src: r.result });
      if (--pending === 0) renderAssets();
    };
    r.readAsDataURL(f);
  });
  e.target.value = '';
});

/* ---------- Demo Scene ---------- */
function buildDemo() {
  const ground = makeObject('rect');
  ground.name = 'الأرضية';
  ground.x = 0; ground.y = 320;
  ground.w = 900; ground.h = 40;
  ground.color = '#2e6b4f';
  ground.physics = { enabled: true, static: true, gravity: 0, vx: 0, vy: 0 };

  const player = makeObject('rect');
  player.name = 'اللاعب';
  player.x = -100; player.y = -200;
  player.w = 56; player.h = 56;
  player.color = '#4a9eff';
  player.physics = { enabled: true, static: false, gravity: 1600, vx: 0, vy: 0 };
  player.script =
    '// سكربت اللاعب — تحكم بالأسهم\n' +
    'var speed = 320;\n' +
    'var jump = -650;\n' +
    '\n' +
    'if (input["arrowleft"] || input["a"]) obj.physics.vx = -speed;\n' +
    'else if (input["arrowright"] || input["d"]) obj.physics.vx = speed;\n' +
    'else obj.physics.vx *= 0.82;\n' +
    '\n' +
    'if ((input["arrowup"] || input["w"] || input[" "]) && obj.physics.vy > -60) {\n' +
    '  obj.physics.vy = jump;\n' +
    '  input[" "] = false;\n' +
    '  input["arrowup"] = false;\n' +
    '  input["w"] = false;\n' +
    '}\n';

  const box = makeObject('rect');
  box.name = 'صندوق';
  box.x = 90; box.y = -80;
  box.w = 70; box.h = 70;
  box.color = '#ffc857';
  box.physics = { enabled: true, static: false, gravity: 1600, vx: 0, vy: 0 };

  const title = makeObject('text');
  title.name = 'العنوان';
  title.text = '⚡ GameForge';
  title.x = 0; title.y = -330;
  title.fontSize = 54;
  title.color = '#7c5cff';

  const hint = makeObject('text');
  hint.name = 'تعليمات';
  hint.text = 'اضغط تشغيل ثم استخدم الأسهم للحركة';
  hint.x = 0; hint.y = -270;
  hint.fontSize = 18;
  hint.color = '#8a8a96';

  scene.objects = [ground, player, box, title, hint];
}

/* ---------- Init ---------- */
function init() {
  resizeCanvas();
  buildDemo();
  renderHierarchy();
  renderAssets();
  buildInspector();
  $('statZoom').textContent = '100%';
  $('stateDot').className = 'dot off';
  requestAnimationFrame(frame);
}

window.addEventListener('load', init);
setTimeout(resizeCanvas, 60);
</script>
</body>
</html>
```

محرر الألعاب والفيزياء في GameForge

هذا المحرر يمنحك أدوات مرئية ونصية لبناء الألعاب خطوة بخطوة.

· تحرير المشهد: أضف كائنات (مربع، دائرة، نص، صورة) من لوحة المشهد، ثم اسحبها أو غيّر حجمها ودورانها مباشرة على اللوحة.
· الخصائص والفيزياء: اختر أي كائن لتعديل خصائصه من لوحة الخصائص، بما في ذلك تفعيل الفيزياء وتحديد الجاذبية والسرعة.
· السكربتات: يمكنك إرفاق كود JavaScript بكل كائن للتحكم في سلوكه، مع متغيرات جاهزة مثل obj وinput وapi.
· التصدير والاختبار: شغّل اللعبة داخل المحرر، ثم صدّرها كملف HTML مستقل يعمل في أي متصفح.

---

Optimization Tip: يمكنك تعديل مشهد البداية من دالة buildDemo() لتغيير الكائنات أو إزالة السكربت التجريبي للاعب.
