# Sulieman<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, user-scalable=no">
<title>نظارتي الذكية برو</title>
<style>
  :root { --blue:#1E5BFF; --navy:#0B1B4D; --ice:#BFD4FF; --glass:rgba(11,27,77,.7); --line:rgba(191,212,255,.3); }
  * { box-sizing:border-box; margin:0; }
  html,body { height:100%; background:#050B24; color:#fff; overflow:hidden; font-family:'Segoe UI',Tahoma,system-ui,sans-serif; }
  #view { position:fixed; inset:0; width:100%; height:100%; touch-action:none; display:block; }
  #cam { position:fixed; width:2px; height:2px; opacity:0; pointer-events:none; }
  .bar { position:fixed; top:0; inset-inline:0; display:flex; gap:8px; padding:calc(env(safe-area-inset-top) + 8px) 10px 8px;
    overflow-x:auto; scrollbar-width:none; z-index:5; }
  .bar::-webkit-scrollbar { display:none; }
  .bar.hide { display:none; }
  .pill { flex:0 0 auto; border:1px solid var(--line); background:var(--glass); color:#fff; border-radius:20px; padding:8px 14px;
    font-size:13px; cursor:pointer; backdrop-filter:blur(10px); white-space:nowrap; }
  .pill.on { background:var(--blue); border-color:var(--blue); }
  .pill:active { transform:scale(.96); }
  .overlay { position:fixed; inset:0; z-index:10; display:flex; align-items:center; justify-content:center; padding:20px; overflow-y:auto;
    background:radial-gradient(circle at 50% 20%, #17398f, #050B24 70%); }
  .overlay[hidden] { display:none; }
  .panel { width:100%; max-width:460px; text-align:center; display:flex; flex-direction:column; gap:14px; }
  .panel h1 { font-size:28px; }
  .panel p { color:var(--ice); line-height:1.7; font-size:15px; }
  .steps { text-align:start; background:rgba(255,255,255,.06); border:1px solid var(--line); border-radius:18px; padding:14px 18px;
    display:flex; flex-direction:column; gap:8px; font-size:14px; line-height:1.6; }
  .big { border:none; background:var(--blue); color:#fff; font-size:19px; font-weight:700; padding:15px; border-radius:16px; cursor:pointer; }
  input.k { width:100%; border:1px solid var(--line); background:rgba(255,255,255,.08); color:#fff; border-radius:12px; padding:13px; font-size:15px; outline:none; direction:ltr; }
  button:focus-visible,input:focus-visible { outline:2px solid var(--ice); outline-offset:2px; }
  @media (prefers-reduced-motion:reduce){ * { animation:none !important; } }
</style>
</head>
<body>
<canvas id="view"></canvas>
<video id="cam" playsinline muted autoplay></video>

<div class="bar" id="bar">
  <button class="pill" id="bStereo"></button>
  <button class="pill" id="bNow"></button>
  <button class="pill" id="bVoice"></button>
  <button class="pill" id="bPause"></button>
  <button class="pill" id="bFlip"></button>
  <button class="pill" id="bLang"></button>
  <button class="pill" id="bKey"></button>
</div>

<div class="overlay" id="start">
  <div class="panel">
    <h1 id="sTitle"></h1>
    <p id="sSub"></p>
    <div class="steps" id="sSteps"></div>
    <button class="big" id="go"></button>
    <button class="pill" id="sLang" style="align-self:center"></button>
  </div>
</div>

<div class="overlay" id="settings" hidden>
  <div class="panel">
    <h1 id="kTitle"></h1>
    <p id="kNote"></p>
    <input class="k" id="keyIn" type="password" placeholder="sk-ant-..." autocomplete="off">
    <button class="big" id="keySave"></button>
  </div>
</div>

<script type="module">
const STR = {
  ar: {
    title:"نظارتي الذكية برو", sub:"تحلل كل شيء أمامك تلقائياً، وتحرّك المعلومات بيدك في الهواء.",
    steps:["١. ضع الجوال أفقياً في حامل رأس (مثل Cardboard) وفعّل «عدسة النظارة».","٢. وجّه الكاميرا للأمام، ستظهر بطاقات على الأشياء تلقائياً.","٣. اقرص بإبهامك وسبابتك على بطاقة وحرّكها وأفلتها.","٤. كفّ مفتوح ثانية وربع = حلّل الآن. قبضة = امسح البطاقات."],
    go:"ابدأ", stereo:"عدسة النظارة", mono:"شاشة عادية", now:"حلّل الآن", voiceOn:"🔊 الصوت: يعمل", voiceOff:"🔇 الصوت: متوقف",
    pause:"إيقاف", resume:"استئناف", flip:"قلب الكاميرا", lang:"EN", key:"مفتاح API",
    kTitle:"مفتاح API", kNote:"لتجربة شخصية فقط: الصق مفتاحك من console.anthropic.com. يبقى على جهازك. للنشر العام نحتاج سيرفر يحمل المفتاح.", save:"حفظ",
    analyzing:"يحلل...", live:"مباشر", paused:"متوقف مؤقتاً", errKey:"أدخل مفتاح API من الإعدادات", errNet:"تعذر الاتصال، سأعيد المحاولة",
    noCam:"الكاميرا غير مفعّلة. اسمح بالإذن وافتح الصفحة عبر https.", handsLoading:"جاري تحميل تتبع اليد", handsReady:"اليد جاهزة", handsFailed:"تتبع اليد غير متاح، استخدم اللمس",
    hint:"اقرص على أي بطاقة وحرّكها • كفّ مفتوح = حلّل الآن • قبضة = امسح", exit:"اضغط مرتين على الشاشة للخروج من وضع النظارة", text:"نص",
    system:`أنت نظارة ذكية يرتديها المستخدم، ترى ما يراه عبر الكاميرا وتحلل أي شيء أمامه. أعد JSON فقط بدون أي شرح أو علامات، بهذا الشكل:
{"title":"","summary":"","text":"","objects":[{"label":"","note":"","box":[x,y,w,h]}]}
box نسب من 0 إلى 1 من أبعاد الصورة، (x,y) الزاوية العليا اليسرى. اذكر حتى 6 أشياء مهمة فقط. label كلمة أو كلمتان. note حتى 9 كلمات وتعطي أعلى فائدة: ما هو، فائدته، تحذير، سعر تقريبي، سعرات للأكل، اسم النبات، حل المسألة، إلخ. title حتى 4 كلمات. summary جملة أو جملتان. text: أي نص مقروء في الصورة مترجماً للعربية إن كان بلغة أخرى، وإلا فاتركه فارغاً. اكتب بالعربية.`,
  },
  en: {
    title:"My Smart Glasses Pro", sub:"Analyzes everything in front of you automatically, and lets you move the info with your hands.",
    steps:["1. Put your phone in landscape in a headset holder (Cardboard style) and turn on Glasses lens.","2. Point the camera ahead; cards appear on objects automatically.","3. Pinch a card with thumb and index finger, move it, and let go.","4. Open palm for 1.2s = analyze now. Fist = clear cards."],
    go:"Start", stereo:"Glasses lens", mono:"Normal screen", now:"Analyze now", voiceOn:"🔊 Voice: on", voiceOff:"🔇 Voice: off",
    pause:"Pause", resume:"Resume", flip:"Flip camera", lang:"عربي", key:"API key",
    kTitle:"API key", kNote:"For personal testing only: paste your key from console.anthropic.com. It stays on your device. For a public launch you need a server that holds the key.", save:"Save",
    analyzing:"Analyzing...", live:"Live", paused:"Paused", errKey:"Enter your API key in settings", errNet:"Connection failed, retrying",
    noCam:"Camera is off. Allow access and open the page over https.", handsLoading:"Loading hand tracking", handsReady:"Hands ready", handsFailed:"Hand tracking unavailable, use touch",
    hint:"Pinch any card and move it • Open palm = analyze now • Fist = clear", exit:"Double-tap the screen to leave glasses mode", text:"Text",
    system:`You are smart glasses worn by the user. You see what they see through the camera and analyze anything in front of them. Reply with JSON only, no explanation or fences, in this shape:
{"title":"","summary":"","text":"","objects":[{"label":"","note":"","box":[x,y,w,h]}]}
box values are fractions 0 to 1 of the image size, (x,y) is the top-left corner. List at most 6 important things. label is one or two words. note is up to 9 words giving the most useful fact: what it is, what it does, a warning, a rough price, calories for food, plant name, the solution to a problem, and so on. title up to 4 words. summary one or two sentences. text: any readable text in the image translated to English if it is in another language, otherwise empty. Write in English.`,
  },
};

const $ = (id) => document.getElementById(id);
const cv = $("view"), ctx = cv.getContext("2d"), video = $("cam");
let lang = "ar", facing = "environment", mirror = false, stereo = false, voiceOn = false, paused = false;
let started = false, camOk = true, stream = null, landmarker = null, handStatus = "idle";
let apiKey = ""; try { apiKey = localStorage.getItem("sg_key") || ""; } catch (e) {}
let W = 0, H = 0, dpr = 1;
const S = () => STR[lang];
const rtl = () => lang === "ar";

/* ---------- layout helpers ---------- */
function resize() { dpr = Math.min(devicePixelRatio || 1, 2); W = innerWidth; H = innerHeight; cv.width = W * dpr; cv.height = H * dpr; }
addEventListener("resize", resize); resize();
const eye = () => (stereo ? { ew: W / 2, eh: H } : { ew: W, eh: H });
const scaleOf = (ew, eh) => Math.max(0.72, Math.min(1.3, Math.min(ew, eh * 1.1) / 430));
function vToE(nx, ny, ew, eh) {
  const vw = video.videoWidth || 16, vh = video.videoHeight || 9, k = Math.max(ew / vw, eh / vh), dw = vw * k, dh = vh * k;
  return { x: (ew - dw) / 2 + (mirror ? 1 - nx : nx) * dw, y: (eh - dh) / 2 + ny * dh };
}
function rr(x, y, w, h, r) {
  ctx.beginPath(); ctx.moveTo(x + r, y); ctx.arcTo(x + w, y, x + w, y + h, r); ctx.arcTo(x + w, y + h, x, y + h, r);
  ctx.arcTo(x, y + h, x, y, r); ctx.arcTo(x, y, x + w, y, r); ctx.closePath();
}
function wrap(text, maxW, maxLines) {
  const words = String(text || "").split(/\s+/).filter(Boolean), lines = []; let cur = "";
  for (const w of words) {
    const t = cur ? cur + " " + w : w;
    if (ctx.measureText(t).width > maxW && cur) { lines.push(cur); cur = w; } else cur = t;
  }
  if (cur) lines.push(cur);
  if (lines.length > maxLines) { lines.length = maxLines; lines[maxLines - 1] = lines[maxLines - 1].replace(/\s*\S*$/, "") + "…"; }
  return lines;
}
const font = (n, w = 500) => `${w} ${Math.round(n)}px "Segoe UI", Tahoma, system-ui, sans-serif`;

/* ---------- state ---------- */
let cards = [], scene = { title: "", summary: "", text: "" };
let status = "idle", busy = false, lastAnalysis = 0, lastSig = null, lastSpoken = "";
let handPts = [], hands21 = [], touchPts = {}, downPrev = {}, grabs = {};
let palmHold = 0, fistHold = 0, gestureLock = false, startedAt = 0, everGrabbed = false, lastT = -1;

/* ---------- camera ---------- */
async function startCam() {
  stream?.getTracks().forEach((t) => t.stop());
  try {
    stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: facing, width: { ideal: 1280 }, height: { ideal: 720 } }, audio: false });
    video.srcObject = stream; await video.play(); mirror = facing === "user"; camOk = true;
  } catch (e) { camOk = false; }
}

/* ---------- hand tracking (MediaPipe) ---------- */
async function loadHands() {
  handStatus = "loading";
  try {
    const base = "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.14";
    const m = await import(base + "/vision_bundle.mjs");
    const fs = await m.FilesetResolver.forVisionTasks(base + "/wasm");
    const make = (d) => m.HandLandmarker.createFromOptions(fs, {
      baseOptions: { modelAssetPath: "https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task", delegate: d },
      runningMode: "VIDEO", numHands: 2,
    });
    try { landmarker = await make("GPU"); } catch (e) { landmarker = await make("CPU"); }
    handStatus = "ready";
  } catch (e) { handStatus = "failed"; }
}
const d2 = (a, b) => Math.hypot(a.x - b.x, a.y - b.y);
const smoothMap = {};
function processHands() {
  if (!landmarker || video.readyState < 2 || video.currentTime === lastT) return;
  lastT = video.currentTime;
  let res; try { res = landmarker.detectForVideo(video, performance.now()); } catch (e) { return; }
  const { ew, eh } = eye();
  handPts = []; hands21 = [];
  (res.landmarks || []).forEach((lm, i) => {
    const size = d2(lm[0], lm[9]) || 0.1, pinch = d2(lm[4], lm[8]) / size;
    const raw = vToE((lm[4].x + lm[8].x) / 2, (lm[4].y + lm[8].y) / 2, ew, eh);
    const id = "h" + i, prev = smoothMap[id];
    const x = prev ? prev.x + (raw.x - prev.x) * 0.55 : raw.x, y = prev ? prev.y + (raw.y - prev.y) * 0.55 : raw.y;
    const down = prev && prev.down ? pinch < 0.55 : pinch < 0.38;
    smoothMap[id] = { x, y, down };
    let ext = 0;
    [[8, 6], [12, 10], [16, 14], [20, 18]].forEach(([t, p]) => { if (d2(lm[t], lm[0]) > d2(lm[p], lm[0]) * 1.08) ext++; });
    const wr = vToE(lm[0].x, lm[0].y, ew, eh);
    handPts.push({ id, x, y, down, ext, wrist: wr });
    hands21.push(lm.map((q) => vToE(q.x, q.y, ew, eh)));
  });
  Object.keys(smoothMap).forEach((k) => { if (!handPts.find((h) => h.id === k)) delete smoothMap[k]; });
}

/* ---------- analysis ---------- */
const tiny = document.createElement("canvas"); tiny.width = 24; tiny.height = 24;
function signature() {
  if (video.readyState < 2) return null;
  const c = tiny.getContext("2d"); c.drawImage(video, 0, 0, 24, 24);
  const d = c.getImageData(0, 0, 24, 24).data, out = new Uint8Array(576);
  for (let i = 0; i < 576; i++) out[i] = (d[i * 4] + d[i * 4 + 1] + d[i * 4 + 2]) / 3;
  return out;
}
const meanDiff = (a, b) => { let s = 0; for (let i = 0; i < a.length; i++) s += Math.abs(a[i] - b[i]); return s / a.length; };
function grabFrame() {
  const w = video.videoWidth, h = video.videoHeight; if (!w || !h) return null;
  const k = Math.min(1, 1024 / Math.max(w, h)), c = document.createElement("canvas");
  c.width = Math.round(w * k); c.height = Math.round(h * k);
  c.getContext("2d").drawImage(video, 0, 0, c.width, c.height);
  return c.toDataURL("image/jpeg", 0.8).split(",")[1];
}
function num(v) { v = Number(v); return isFinite(v) ? Math.max(0, Math.min(1, v)) : null; }

async function analyze() {
  if (busy || !camOk) return;
  const frame = grabFrame(); if (!frame) return;
  busy = true; status = "analyzing"; const sig = signature();
  try {
    const headers = { "Content-Type": "application/json" };
    if (apiKey) { headers["x-api-key"] = apiKey; headers["anthropic-version"] = "2023-06-01"; headers["anthropic-dangerous-direct-browser-access"] = "true"; }
    const res = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST", headers,
      body: JSON.stringify({ model: "claude-sonnet-4-6", max_tokens: 1000, system: S().system,
        messages: [{ role: "user", content: [{ type: "image", source: { type: "base64", media_type: "image/jpeg", data: frame } },
          { type: "text", text: lang === "ar" ? "حلل ما أراه الآن." : "Analyze what I see now." }] }] }),
    });
    const j = await res.json();
    if (j.error) { const e = new Error(j.error.message || "api"); e.auth = res.status === 401; throw e; }
    const raw = (j.content || []).map((b) => b.text || "").join("");
    const o = JSON.parse(raw.slice(raw.indexOf("{"), raw.lastIndexOf("}") + 1));
    const pinned = cards.filter((c) => c.pinned);
    const fresh = (o.objects || []).slice(0, 6).map((ob, i) => {
      const b = Array.isArray(ob.box) ? ob.box.map(num) : [];
      if (b.length < 4 || b.some((v) => v === null)) return null;
      return { id: Date.now() + i, label: String(ob.label || ""), note: String(ob.note || ""), b, pinned: false, t0: performance.now() };
    }).filter(Boolean);
    cards = [...fresh, ...pinned];
    scene = { title: String(o.title || ""), summary: String(o.summary || ""), text: String(o.text || "") };
    status = "live";
    if (voiceOn && scene.title && scene.title !== lastSpoken) { lastSpoken = scene.title; speak(scene.title + ". " + scene.summary); }
  } catch (e) { status = !apiKey ? "errKey" : e.auth ? "errKey" : "errNet"; }
  finally { busy = false; lastAnalysis = performance.now(); lastSig = sig; }
}
setInterval(() => {
  if (!started || paused || busy || !camOk || Object.keys(grabs).length) return;
  const since = performance.now() - lastAnalysis, sig = signature();
  if (!sig) return;
  const diff = lastSig ? meanDiff(sig, lastSig) : 999;
  if (since > 12000 || (since > 3500 && diff > 14)) analyze();
}, 800);

function speak(text) {
  if (!window.speechSynthesis) return;
  speechSynthesis.cancel();
  const u = new SpeechSynthesisUtterance(text); u.lang = lang === "ar" ? "ar-SA" : "en-US";
  const v = speechSynthesis.getVoices().find((x) => x.lang.toLowerCase().startsWith(lang));
  if (v) u.voice = v; speechSynthesis.speak(u);
}

/* ---------- cards: layout + interaction ---------- */
function layout(c, ew, eh, s) {
  const w = 188 * s, h = 70 * s;
  if (c.pinned) { c.rect = { x: c.px * ew, y: c.py * eh, w, h }; c.box = null; return; }
  const p1 = vToE(c.b[0], c.b[1], ew, eh), p2 = vToE(c.b[0] + c.b[2], c.b[1] + c.b[3], ew, eh);
  const b = { x: Math.min(p1.x, p2.x), y: Math.min(p1.y, p2.y), w: Math.abs(p2.x - p1.x), h: Math.abs(p2.y - p1.y) };
  c.box = b;
  let x = b.x + b.w / 2 - w / 2, y = b.y - h - 12 * s; if (y < 44 * s) y = b.y + b.h + 12 * s;
  c.rect = { x: Math.max(6, Math.min(ew - w - 6, x)), y: Math.max(40 * s, Math.min(eh - h - 6, y)), w, h };
}
function hitCard(x, y) {
  for (let i = cards.length - 1; i >= 0; i--) {
    const c = cards[i], r = c.rect; if (!r) continue;
    if (x >= r.x - 10 && x <= r.x + r.w + 10 && y >= r.y - 10 && y <= r.y + r.h + 10) return c;
    if (c.box && x >= c.box.x && x <= c.box.x + c.box.w && y >= c.box.y && y <= c.box.y + c.box.h) return c;
  }
  return null;
}
function update(dt) {
  processHands();
  const { ew, eh } = eye(), s = scaleOf(ew, eh);
  cards.forEach((c) => layout(c, ew, eh, s));
  const all = [...handPts, ...Object.values(touchPts)], seen = new Set();
  all.forEach((p) => {
    seen.add(p.id);
    if (p.down && !downPrev[p.id]) {
      const c = hitCard(p.x, p.y);
      if (c) {
        c.pinned = true; c.px = c.rect.x / ew; c.py = c.rect.y / eh; c.box = null;
        grabs[p.id] = { c, dx: c.rect.x - p.x, dy: c.rect.y - p.y };
        cards.splice(cards.indexOf(c), 1); cards.push(c); everGrabbed = true;
        const pins = cards.filter((q) => q.pinned); if (pins.length > 8) cards.splice(cards.indexOf(pins[0]), 1);
      }
    }
    const g = grabs[p.id];
    if (g) { if (p.down) { g.c.px = (p.x + g.dx) / ew; g.c.py = (p.y + g.dy) / eh; g.c.rect.x = p.x + g.dx; g.c.rect.y = p.y + g.dy; } else delete grabs[p.id]; }
    downPrev[p.id] = p.down;
  });
  Object.keys(grabs).forEach((id) => { if (!seen.has(id)) delete grabs[id]; });
  Object.keys(downPrev).forEach((id) => { if (!seen.has(id)) delete downPrev[id]; });

  const h0 = handPts[0];
  if (h0 && !h0.down && !Object.keys(grabs).length) {
    palmHold = h0.ext === 4 ? palmHold + dt : 0; fistHold = h0.ext === 0 ? fistHold + dt : 0;
    if (!gestureLock && palmHold > 1.2) { gestureLock = true; analyze(); }
    if (!gestureLock && fistHold > 1.2) { gestureLock = true; cards = []; scene = { title: "", summary: "", text: "" }; }
  } else { palmHold = 0; fistHold = 0; }
  if (!h0 || (h0.ext !== 4 && h0.ext !== 0)) gestureLock = false;
}

/* ---------- drawing ---------- */
const HAND_LINES = [[0,1],[1,2],[2,3],[3,4],[0,5],[5,6],[6,7],[7,8],[5,9],[9,10],[10,11],[11,12],[9,13],[13,14],[14,15],[15,16],[13,17],[17,18],[18,19],[19,20],[0,17]];
function drawScene(ew, eh) {
  const s = scaleOf(ew, eh), now = performance.now(), R = rtl();
  ctx.direction = R ? "rtl" : "ltr"; ctx.textAlign = "start"; ctx.textBaseline = "alphabetic";
  const ax = (x, w, pad) => (R ? x + w - pad : x + pad);

  // video
  if (camOk && video.readyState >= 2) {
    const vw = video.videoWidth, vh = video.videoHeight, k = Math.max(ew / vw, eh / vh), dw = vw * k, dh = vh * k, ox = (ew - dw) / 2, oy = (eh - dh) / 2;
    if (mirror) { ctx.save(); ctx.translate(ew, 0); ctx.scale(-1, 1); ctx.drawImage(video, ox, oy, dw, dh); ctx.restore(); }
    else ctx.drawImage(video, ox, oy, dw, dh);
  } else {
    ctx.fillStyle = "#0B1B4D"; ctx.fillRect(0, 0, ew, eh);
    if (started && !camOk) { ctx.fillStyle = "#BFD4FF"; ctx.font = font(15 * s); ctx.textAlign = "center"; ctx.fillText(S().noCam, ew / 2, eh / 2); ctx.textAlign = "start"; }
  }
  const g = ctx.createRadialGradient(ew / 2, eh / 2, Math.min(ew, eh) * 0.35, ew / 2, eh / 2, Math.max(ew, eh) * 0.8);
  g.addColorStop(0, "rgba(5,11,36,0)"); g.addColorStop(1, "rgba(5,11,36,.6)"); ctx.fillStyle = g; ctx.fillRect(0, 0, ew, eh);
  if (!started) return;

  // cards
  cards.forEach((c) => {
    const r = c.rect; if (!r) return;
    const a = c.pinned ? 1 : Math.min(1, (now - c.t0) / 350);
    ctx.globalAlpha = a;
    if (c.box) {
      const b = c.box, L = Math.min(18 * s, b.w / 3, b.h / 3);
      ctx.strokeStyle = "#BFD4FF"; ctx.lineWidth = 2.2; ctx.beginPath();
      [[b.x, b.y, 1, 1], [b.x + b.w, b.y, -1, 1], [b.x, b.y + b.h, 1, -1], [b.x + b.w, b.y + b.h, -1, -1]].forEach(([x, y, dx, dy]) => {
        ctx.moveTo(x + dx * L, y); ctx.lineTo(x, y); ctx.lineTo(x, y + dy * L);
      }); ctx.stroke();
      ctx.strokeStyle = "rgba(191,212,255,.5)"; ctx.lineWidth = 1.2; ctx.beginPath();
      const up = r.y < b.y; ctx.moveTo(r.x + r.w / 2, up ? r.y + r.h : r.y); ctx.lineTo(b.x + b.w / 2, up ? b.y : b.y + b.h); ctx.stroke();
    }
    const grabbed = Object.values(grabs).some((q) => q.c === c);
    ctx.save();
    if (grabbed || c.pinned) { ctx.shadowColor = grabbed ? "#7FB0FF" : "rgba(30,91,255,.6)"; ctx.shadowBlur = grabbed ? 24 : 10; }
    rr(r.x, r.y, r.w, r.h, 14 * s); ctx.fillStyle = "rgba(11,27,77,.82)"; ctx.fill(); ctx.restore();
    rr(r.x, r.y, r.w, r.h, 14 * s); ctx.strokeStyle = grabbed ? "#fff" : c.pinned ? "#7FB0FF" : "rgba(191,212,255,.45)"; ctx.lineWidth = grabbed ? 2.4 : 1.4; ctx.stroke();
    ctx.fillStyle = "#1E5BFF"; ctx.beginPath(); ctx.arc(ax(r.x, r.w, 14 * s), r.y + 18 * s, 4.5 * s, 0, 7); ctx.fill();
    const tx = ax(r.x, r.w, 26 * s);
    ctx.font = font(15 * s, 700); ctx.fillStyle = "#fff";
    ctx.fillText(wrap(c.label, r.w - 40 * s, 1)[0] || "", tx, r.y + 23 * s);
    ctx.font = font(12.5 * s); ctx.fillStyle = "#BFD4FF";
    wrap(c.note, r.w - 24 * s, 2).forEach((ln, i) => ctx.fillText(ln, ax(r.x, r.w, 12 * s), r.y + (42 + i * 16) * s));
    ctx.globalAlpha = 1;
  });

  // hands
  hands21.forEach((pts) => {
    ctx.strokeStyle = "rgba(191,212,255,.55)"; ctx.lineWidth = 2; ctx.beginPath();
    HAND_LINES.forEach(([a, b]) => { ctx.moveTo(pts[a].x, pts[a].y); ctx.lineTo(pts[b].x, pts[b].y); }); ctx.stroke();
    ctx.fillStyle = "#fff"; pts.forEach((p) => { ctx.beginPath(); ctx.arc(p.x, p.y, 2.6, 0, 7); ctx.fill(); });
  });
  const cursors = [...handPts, ...Object.values(touchPts)];
  cursors.forEach((p) => {
    ctx.beginPath(); ctx.arc(p.x, p.y, (p.down ? 9 : 14) * s, 0, 7);
    ctx.fillStyle = p.down ? "rgba(30,91,255,.9)" : "rgba(255,255,255,.12)"; ctx.fill();
    ctx.strokeStyle = "#fff"; ctx.lineWidth = 2.2; ctx.stroke();
  });
  const h0 = handPts[0];
  if (h0 && (palmHold > 0.15 || fistHold > 0.15)) {
    const prog = Math.min(1, (palmHold || fistHold) / 1.2);
    ctx.beginPath(); ctx.arc(h0.wrist.x, h0.wrist.y, 26 * s, -Math.PI / 2, -Math.PI / 2 + prog * Math.PI * 2);
    ctx.strokeStyle = palmHold ? "#3DDC97" : "#FF4D6A"; ctx.lineWidth = 5; ctx.stroke();
  }

  // status
  const label = paused ? S().paused : status === "analyzing" ? S().analyzing : status === "live" ? S().live : status === "errKey" ? S().errKey : status === "errNet" ? S().errNet : "";
  const hs = handStatus === "loading" ? S().handsLoading : handStatus === "failed" ? S().handsFailed : handStatus === "ready" ? S().handsReady + (handPts.length ? " ✋" + handPts.length : "") : "";
  ctx.font = font(12.5 * s, 600);
  const dotC = status.startsWith("err") ? "#FF4D6A" : status === "analyzing" ? "#FFC857" : "#3DDC97";
  const sx = ax(0, ew, 12 * s), top = (stereo ? 22 : 54) * s;
  ctx.fillStyle = dotC; ctx.beginPath(); ctx.arc(R ? sx - 4 : sx + 4, top - 4, 4, 0, 7); ctx.fill();
  ctx.fillStyle = "#fff"; ctx.fillText(label, R ? sx - 14 : sx + 14, top);
  ctx.fillStyle = "#BFD4FF"; ctx.font = font(11.5 * s); ctx.fillText(hs, sx, top + 17 * s);

  // summary HUD
  if (scene.title) {
    const pw = Math.min(ew - 24, 540 * s), x = (ew - pw) / 2;
    ctx.font = font(13.5 * s); const lines = wrap(scene.summary, pw - 30 * s, 3);
    const tl = scene.text ? wrap(S().text + ": " + scene.text, pw - 30 * s, 2) : [];
    const ph = (30 + lines.length * 18 + (tl.length ? tl.length * 17 + 8 : 0) + 8) * s, y = eh - ph - 14 * s;
    rr(x, y, pw, ph, 18 * s); ctx.fillStyle = "rgba(11,27,77,.76)"; ctx.fill(); ctx.strokeStyle = "rgba(191,212,255,.35)"; ctx.lineWidth = 1.2; ctx.stroke();
    ctx.font = font(15.5 * s, 700); ctx.fillStyle = "#fff"; ctx.fillText(scene.title, ax(x, pw, 16 * s), y + 23 * s);
    ctx.font = font(13.5 * s); ctx.fillStyle = "#DCE7FF"; lines.forEach((ln, i) => ctx.fillText(ln, ax(x, pw, 16 * s), y + (43 + i * 18) * s));
    if (tl.length) { ctx.font = font(12.5 * s); ctx.fillStyle = "#9CC4FF"; tl.forEach((ln, i) => ctx.fillText(ln, ax(x, pw, 16 * s), y + (43 + lines.length * 18 + 8 + i * 17) * s)); }
  }

  // hint
  const age = (now - startedAt) / 1000;
  if (!everGrabbed && age < 25 && cards.length) {
    ctx.textAlign = "center"; ctx.font = font(13 * s, 600); const t = stereo && age < 6 ? S().exit : S().hint;
    const tw = Math.min(ew - 20, ctx.measureText(t).width + 28);
    rr((ew - tw) / 2, 78 * s, tw, 30 * s, 15 * s); ctx.fillStyle = "rgba(30,91,255,.85)"; ctx.fill();
    ctx.fillStyle = "#fff"; ctx.fillText(t, ew / 2, 98 * s); ctx.textAlign = "start";
  }
}

let lastFrame = performance.now();
function frame(now) {
  const dt = Math.min(0.1, (now - lastFrame) / 1000); lastFrame = now;
  if (started) update(dt);
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0); ctx.clearRect(0, 0, W, H);
  const { ew, eh } = eye();
  for (let e = 0; e < (stereo ? 2 : 1); e++) {
    ctx.save(); ctx.beginPath(); ctx.rect(e * ew, 0, ew, eh); ctx.clip(); ctx.translate(e * ew, 0); drawScene(ew, eh); ctx.restore();
  }
  if (stereo) { ctx.fillStyle = "#000"; ctx.fillRect(W / 2 - 1.5, 0, 3, H); }
  requestAnimationFrame(frame);
}
requestAnimationFrame(frame);

/* ---------- pointer (touch / mouse fallback) ---------- */
let lastTap = 0;
function toEye(e) { const { ew } = eye(); return { x: stereo ? e.clientX % ew : e.clientX, y: e.clientY }; }
cv.addEventListener("pointerdown", (e) => {
  const n = performance.now(); if (stereo && n - lastTap < 350) { setStereo(false); lastTap = 0; return; } lastTap = n;
  cv.setPointerCapture(e.pointerId); touchPts[e.pointerId] = { id: "t" + e.pointerId, ...toEye(e), down: true };
});
cv.addEventListener("pointermove", (e) => { const p = touchPts[e.pointerId]; if (p) Object.assign(p, toEye(e)); });
["pointerup", "pointercancel"].forEach((t) => cv.addEventListener(t, (e) => { delete touchPts[e.pointerId]; }));

/* ---------- UI ---------- */
function setStereo(v) { stereo = v; $("bar").classList.toggle("hide", v); refreshUI(); if (v) startedAt = performance.now(); }
function refreshUI() {
  const t = S();
  document.documentElement.lang = lang; document.documentElement.dir = rtl() ? "rtl" : "ltr";
  $("sTitle").textContent = t.title; $("sSub").textContent = t.sub; $("go").textContent = t.go; $("sLang").textContent = t.lang;
  $("sSteps").innerHTML = ""; t.steps.forEach((x) => { const d = document.createElement("div"); d.textContent = x; $("sSteps").appendChild(d); });
  $("bStereo").textContent = stereo ? t.mono : t.stereo; $("bStereo").classList.toggle("on", stereo);
  $("bNow").textContent = t.now; $("bVoice").textContent = voiceOn ? t.voiceOn : t.voiceOff; $("bVoice").classList.toggle("on", voiceOn);
  $("bPause").textContent = paused ? t.resume : t.pause; $("bPause").classList.toggle("on", paused);
  $("bFlip").textContent = t.flip; $("bLang").textContent = t.lang; $("bKey").textContent = t.key;
  $("kTitle").textContent = t.kTitle; $("kNote").textContent = t.kNote; $("keySave").textContent = t.save;
}
$("go").onclick = async () => {
  $("start").hidden = true; started = true; startedAt = performance.now();
  try { await document.documentElement.requestFullscreen?.(); } catch (e) {}
  try { await screen.orientation?.lock?.("landscape"); } catch (e) {}
  try { await navigator.wakeLock?.request("screen"); } catch (e) {}
  await startCam(); loadHands();
};
$("sLang").onclick = $("bLang").onclick = () => { lang = lang === "ar" ? "en" : "ar"; cards = cards.filter((c) => c.pinned); scene = { title: "", summary: "", text: "" }; lastAnalysis = 0; lastSig = null; refreshUI(); };
$("bStereo").onclick = () => setStereo(!stereo);
$("bNow").onclick = analyze;
$("bVoice").onclick = () => { voiceOn = !voiceOn; if (voiceOn) speak(lang === "ar" ? "تم" : "Okay"); refreshUI(); };
$("bPause").onclick = () => { paused = !paused; refreshUI(); };
$("bFlip").onclick = async () => { facing = facing === "environment" ? "user" : "environment"; await startCam(); };
$("bKey").onclick = () => { $("keyIn").value = apiKey; $("settings").hidden = false; };
$("keySave").onclick = () => { apiKey = $("keyIn").value.trim(); try { localStorage.setItem("sg_key", apiKey); } catch (e) {} $("settings").hidden = true; lastAnalysis = 0; };
refreshUI();
</script>
</body>
</html>
