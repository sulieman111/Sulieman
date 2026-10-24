<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, user-scalable=no">
<title>نظارتي الذكية AR</title>
<style>
  :root { --blue:#1E5BFF; --navy:#0B1B4D; --ice:#BFD4FF; --glass:rgba(11,27,77,.78); --line:rgba(191,212,255,.35); }
  * { box-sizing:border-box; margin:0; }
  html,body { height:100%; background:#050B24; color:#fff; overflow:hidden; font-family:'Segoe UI',Tahoma,system-ui,sans-serif; }
  #eyes { position:fixed; inset:0; display:flex; direction:ltr; touch-action:none; user-select:none; -webkit-user-select:none; }
  .eye { position:relative; flex:1; overflow:hidden; background:#050B24; }
  .eye[hidden] { display:none; }
  .eye + .eye { border-inline-start:3px solid #000; }
  .cam { position:absolute; inset:0; width:100%; height:100%; pointer-events:none; }
  .ar { position:absolute; inset:0; }
  .ar.ghost > .win, .ar.ghost > .toasts { opacity:0; pointer-events:none; }
  #vid { position:fixed; width:2px; height:2px; opacity:0; pointer-events:none; }

  /* floating windows */
  .win { position:absolute; background:var(--glass); backdrop-filter:blur(14px); -webkit-backdrop-filter:blur(14px);
    border:1px solid var(--line); border-radius:1.1em; overflow:hidden; box-shadow:0 10px 30px rgba(0,0,0,.4); }
  .hd { display:flex; align-items:center; justify-content:space-between; gap:.5em; padding:.35em .6em; background:rgba(30,91,255,.4); cursor:grab; }
  .tt { font-weight:700; font-size:1em; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
  .bt { display:flex; gap:.3em; direction:ltr; flex:0 0 auto; }
  .bt button, .ctl button { min-width:2.1em; height:2.1em; border:none; border-radius:50%; background:rgba(255,255,255,.16); color:#fff; font-size:1em; cursor:pointer; }
  .bd { padding:.6em; }
  .rz { position:absolute; right:0; bottom:0; width:2.4em; height:2.4em; background:linear-gradient(135deg,transparent 52%,var(--ice) 52%); border-bottom-right-radius:1.1em; opacity:.9; }
  .pl { aspect-ratio:16/9; background:#000; border-radius:.6em; overflow:hidden; }
  .pl iframe { width:100%; height:100%; display:block; border:0; pointer-events:none; }
  .ctl { display:flex; justify-content:center; gap:.5em; margin-top:.5em; direction:ltr; }
  .grid { display:grid; grid-template-columns:1fr 1fr; gap:.5em; }
  .vid { background:rgba(255,255,255,.08); border-radius:.7em; overflow:hidden; cursor:pointer; }
  .vid img { width:100%; aspect-ratio:16/9; object-fit:cover; display:block; pointer-events:none; }
  .vid span { display:block; padding:.3em .5em; font-size:.8em; line-height:1.4; max-height:2.9em; overflow:hidden; pointer-events:none; }
  .muted { color:var(--ice); font-size:.9em; line-height:1.7; }
  .tabs { display:flex; gap:.4em; margin-bottom:.5em; flex-wrap:wrap; }
  .tab { padding:.35em .8em; border-radius:1em; background:rgba(255,255,255,.12); font-size:.9em; position:relative; cursor:pointer; }
  .tab.on { background:var(--blue); }
  .tab i { position:absolute; top:-.2em; inset-inline-end:-.2em; background:#FF4D6A; border-radius:50%; min-width:1.3em; height:1.3em; font-style:normal; font-size:.75em; display:flex; align-items:center; justify-content:center; }
  .thread { display:flex; flex-direction:column; gap:.35em; min-height:5em; }
  .m { max-width:82%; padding:.4em .75em; border-radius:1em; line-height:1.5; font-size:.95em; }
  .m.me { align-self:flex-end; background:var(--blue); } .m.them { align-self:flex-start; background:rgba(255,255,255,.16); }
  .rep { display:flex; flex-wrap:wrap; gap:.4em; margin-top:.55em; }
  .rep button { border:1px solid var(--line); background:rgba(255,255,255,.1); color:#fff; border-radius:1em; padding:.45em .9em; font-size:.9em; cursor:pointer; }
  .it { padding:.35em 0; border-bottom:1px solid rgba(191,212,255,.15); line-height:1.5; font-size:.95em; }
  .it b { color:#fff; } .it span { color:var(--ice); }
  .clk { text-align:center; } .clk .big { font-size:2.4em; font-weight:800; letter-spacing:.03em; direction:ltr; }

  /* dock + toasts + cursors */
  .dock { position:absolute; bottom:3%; left:50%; transform:translateX(-50%); display:flex; gap:.5em; padding:.5em; z-index:900;
    background:var(--glass); border:1px solid var(--line); border-radius:1.6em; backdrop-filter:blur(14px); -webkit-backdrop-filter:blur(14px); }
  .dock button { border:none; background:rgba(255,255,255,.1); color:#fff; border-radius:1.2em; padding:.55em .9em; font-size:1em; cursor:pointer; display:flex; flex-direction:column; align-items:center; gap:.15em; min-width:3.6em; }
  .dock button b { font-size:1.3em; font-weight:400; } .dock button span { font-size:.72em; opacity:.85; }
  .dock button.on { background:var(--blue); }
  .toasts { position:absolute; top:3%; left:50%; transform:translateX(-50%); width:56%; display:flex; flex-direction:column; gap:.5em; z-index:950; }
  .toast { display:flex; gap:.7em; align-items:center; padding:.7em .9em; border-radius:1.2em; background:rgba(11,27,77,.92); border:1px solid var(--line);
    box-shadow:0 8px 24px rgba(0,0,0,.45); position:relative; overflow:hidden; animation:slide .35s ease-out; }
  .toast .av { width:2.4em; height:2.4em; border-radius:50%; display:flex; align-items:center; justify-content:center; font-weight:800; flex:0 0 auto; }
  .toast .tx { min-width:0; } .toast .tx b { display:block; } .toast .tx span { display:block; color:var(--ice); font-size:.92em; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
  .toast .pb { position:absolute; bottom:0; inset-inline-start:0; height:3px; background:#7FB0FF; animation:shrink 7s linear forwards; }
  @keyframes slide { from { transform:translateY(-30px); opacity:0; } to { transform:none; opacity:1; } }
  @keyframes shrink { from { width:100%; } to { width:0; } }
  .cur { position:absolute; left:0; top:0; width:28px; height:28px; margin:-14px 0 0 -14px; border-radius:50%; border:2.5px solid #fff; background:rgba(255,255,255,.14);
    z-index:1000; pointer-events:none; display:none; }
  .cur.down { background:rgba(30,91,255,.9); transform-origin:center; width:20px; height:20px; margin:-10px 0 0 -10px; }
  .ring { position:absolute; width:64px; height:64px; margin:-32px 0 0 -32px; border-radius:50%; z-index:999; pointer-events:none; display:none;
    -webkit-mask:radial-gradient(circle,transparent 56%,#000 58%); mask:radial-gradient(circle,transparent 56%,#000 58%); }

  /* DOM chrome */
  .bar { position:fixed; top:0; inset-inline:0; display:flex; gap:8px; padding:calc(env(safe-area-inset-top) + 8px) 10px 8px; overflow-x:auto; scrollbar-width:none; z-index:20; direction:inherit; }
  .bar::-webkit-scrollbar { display:none; } .bar.hide { display:none; }
  .pill { flex:0 0 auto; border:1px solid var(--line); background:var(--glass); color:#fff; border-radius:20px; padding:8px 14px; font-size:13px; cursor:pointer; backdrop-filter:blur(10px); white-space:nowrap; }
  .pill.on { background:var(--blue); border-color:var(--blue); }
  .overlay { position:fixed; inset:0; z-index:50; display:flex; align-items:center; justify-content:center; padding:20px; overflow-y:auto; background:radial-gradient(circle at 50% 20%,#17398f,#050B24 70%); }
  .overlay[hidden] { display:none; }
  .panel { width:100%; max-width:480px; text-align:center; display:flex; flex-direction:column; gap:14px; }
  .panel h1 { font-size:27px; } .panel p { color:var(--ice); line-height:1.7; font-size:15px; }
  .steps { text-align:start; background:rgba(255,255,255,.06); border:1px solid var(--line); border-radius:18px; padding:14px 18px; display:flex; flex-direction:column; gap:8px; font-size:14px; line-height:1.65; }
  .big { border:none; background:var(--blue); color:#fff; font-size:19px; font-weight:700; padding:15px; border-radius:16px; cursor:pointer; }
  .f { width:100%; border:1px solid var(--line); background:rgba(255,255,255,.08); color:#fff; border-radius:12px; padding:12px; font-size:14px; outline:none; direction:ltr; font-family:inherit; }
  textarea.f { min-height:110px; resize:vertical; } label { text-align:start; font-size:13px; color:var(--ice); }
  button:focus-visible { outline:2px solid var(--ice); outline-offset:2px; }
  @media (prefers-reduced-motion:reduce){ * { animation:none !important; } }
</style>
</head>
<body>
<div id="eyes">
  <div class="eye" id="eye0"><canvas class="cam"></canvas><div class="ar"></div></div>
  <div class="eye" id="eye1" hidden><canvas class="cam"></canvas><div class="ar"></div></div>
</div>
<video id="vid" playsinline muted autoplay></video>

<div class="bar" id="bar">
  <button class="pill" id="bStereo"></button><button class="pill" id="bNow"></button><button class="pill" id="bVoice"></button>
  <button class="pill" id="bPause"></button><button class="pill" id="bDemo"></button><button class="pill" id="bMsg"></button>
  <button class="pill" id="bFlip"></button><button class="pill" id="bLang"></button><button class="pill" id="bSet"></button>
</div>

<div class="overlay" id="start"><div class="panel">
  <h1 id="sTitle"></h1><p id="sSub"></p><div class="steps" id="sSteps"></div>
  <button class="big" id="go"></button><button class="pill" id="sLang" style="align-self:center"></button>
</div></div>

<div class="overlay" id="settings" hidden><div class="panel">
  <h1 id="kTitle"></h1><p id="kNote"></p>
  <label id="kl1"></label><input class="f" id="keyIn" type="password" placeholder="sk-ant-..." autocomplete="off">
  <label id="kl2"></label><textarea class="f" id="linksIn" placeholder="https://www.youtube.com/watch?v=..."></textarea>
  <button class="big" id="keySave"></button>
</div></div>

<script type="module">
/* =================== strings =================== */
const STR = {
  ar: {
    title:"نظارتي الذكية AR", sub:"واقع معزز أمامك: نوافذ تتحكم فيها بيدك، رسائل تظهر على الشاشة، وتحليل تلقائي لكل ما تراه.",
    steps:["١. ابدأ، ثم اضغط «عدسة» وضع الجوال أفقياً في حامل رأس (Cardboard).","٢. اقرص بإبهامك وسبابتك على أي نافذة وحرّكها في الهواء.","٣. لتكبير النافذة وتصغيرها: اقرص + أو −، أو اسحب الزاوية السفلية، أو اقرصها بيديك الاثنتين وباعد بينهما أو قرّبهما.","٤. من الشريط السفلي أضف يوتيوب أو الرسائل أو التحليل أو الساعة.","٥. كفّ مفتوح ثانية وربع = حلّل الآن. قبضة = إخفاء أو إظهار كل النوافذ."],
    go:"ابدأ", stereo:"عدسة النظارة", mono:"شاشة عادية", now:"حلّل الآن", voiceOn:"🔊 الصوت: يعمل", voiceOff:"🔇 الصوت: متوقف",
    pause:"إيقاف التحليل", resume:"استئناف التحليل", demoOn:"رسائل تجريبية: تعمل", demoOff:"رسائل تجريبية: متوقفة", msg:"رسالة الآن",
    flip:"قلب الكاميرا", lang:"EN", settings:"الإعدادات", kNote:"للتجربة الشخصية: مفتاحك يبقى على جهازك. للنشر العام نحتاج سيرفر.",
    kl1:"مفتاح API (من console.anthropic.com)", kl2:"روابط يوتيوب (رابط في كل سطر) — جهّزها قبل لبس الجوال", save:"حفظ",
    dock:{ youtube:"يوتيوب", chat:"الرسائل", analysis:"التحليل", clock:"الساعة", stereo:"عدسة" },
    wt:{ youtube:"يوتيوب", chat:"الرسائل", analysis:"التحليل", clock:"الساعة" },
    ytEmpty:"لا توجد فيديوهات محفوظة. أضف روابط يوتيوب من الإعدادات.", ytPick:"اختر فيديو", demoVid:"فيديو تجريبي",
    reply:["👍","تمام","بعدين أكلمك","أنا في الطريق"], noMsg:"لا رسائل بعد", typing:"تحدث الآن...",
    analyzing:"يحلل...", live:"مباشر", paused:"التحليل متوقف", errKey:"أدخل مفتاح API من الإعدادات", errNet:"تعذر الاتصال، سأعيد المحاولة",
    noCam:"الكاميرا غير مفعّلة. اسمح بالإذن وافتح الصفحة عبر https.", handsLoading:"جاري تحميل تتبع اليد", handsReady:"اليد جاهزة", handsFailed:"تتبع اليد غير متاح، استخدم اللمس",
    hint:"اقرص أي نافذة وحرّكها • + و − للحجم • يدان معاً للتكبير • قبضة = إخفاء", noAn:"ننتظر التحليل...", text:"نص",
    msgs:{ a:["وينك؟ نتقابل الليلة؟","شفت المباراة؟","ابعث لي الموقع من فضلك"], s:["صباح الخير 🌞","خلصتي الشغل؟","شكراً على المساعدة"], m:["الله يحفظك يا ولدي","اتصل علينا لمن تفضى","أكلت؟"] },
    names:{ a:"أحمد", s:"سارة", m:"أمي" }, welcome:"مرحباً! هذه رسالة تجريبية 👋",
    system:`أنت نظارة ذكية يرتديها المستخدم، ترى ما يراه عبر الكاميرا وتحلل أي شيء أمامه. أعد JSON فقط بدون أي شرح أو علامات، بهذا الشكل:
{"title":"","summary":"","text":"","objects":[{"label":"","note":"","box":[x,y,w,h]}]}
box نسب من 0 إلى 1 من أبعاد الصورة، (x,y) الزاوية العليا اليسرى. اذكر حتى 6 أشياء مهمة فقط. label كلمة أو كلمتان. note حتى 9 كلمات وتعطي أعلى فائدة: ما هو، فائدته، تحذير، سعر تقريبي، سعرات للأكل، اسم النبات، حل المسألة، إلخ. title حتى 4 كلمات. summary جملة أو جملتان. text: أي نص مقروء في الصورة مترجماً للعربية إن كان بلغة أخرى، وإلا فاتركه فارغاً. اكتب بالعربية.`,
  },
  en: {
    title:"My Smart Glasses AR", sub:"Augmented reality in front of you: windows you control with your hands, messages that pop up on screen, and automatic analysis of everything you see.",
    steps:["1. Tap Start, then Glasses lens, and put your phone in landscape in a headset holder (Cardboard).","2. Pinch any window with thumb and index finger and move it in the air.","3. To enlarge or shrink a window: pinch + or −, drag the bottom corner, or pinch it with both hands and move them apart or together.","4. Use the bottom dock to add YouTube, Messages, Analysis or Clock.","5. Open palm for 1.2s = analyze now. Fist = hide or show all windows."],
    go:"Start", stereo:"Glasses lens", mono:"Normal screen", now:"Analyze now", voiceOn:"🔊 Voice: on", voiceOff:"🔇 Voice: off",
    pause:"Pause analysis", resume:"Resume analysis", demoOn:"Demo messages: on", demoOff:"Demo messages: off", msg:"Message now",
    flip:"Flip camera", lang:"عربي", settings:"Settings", kNote:"For personal testing: your key stays on your device. A public launch needs a server.",
    kl1:"API key (from console.anthropic.com)", kl2:"YouTube links (one per line) — prepare them before wearing the phone", save:"Save",
    dock:{ youtube:"YouTube", chat:"Messages", analysis:"Analysis", clock:"Clock", stereo:"Lens" },
    wt:{ youtube:"YouTube", chat:"Messages", analysis:"Analysis", clock:"Clock" },
    ytEmpty:"No saved videos. Add YouTube links in Settings.", ytPick:"Pick a video", demoVid:"Demo video",
    reply:["👍","Okay","Talk later","On my way"], noMsg:"No messages yet", typing:"Speak now...",
    analyzing:"Analyzing...", live:"Live", paused:"Analysis paused", errKey:"Enter your API key in Settings", errNet:"Connection failed, retrying",
    noCam:"Camera is off. Allow access and open the page over https.", handsLoading:"Loading hand tracking", handsReady:"Hands ready", handsFailed:"Hand tracking unavailable, use touch",
    hint:"Pinch a window and move it • + and − resize • two hands zoom • fist = hide", noAn:"Waiting for analysis...", text:"Text",
    msgs:{ a:["Where are you? Meet tonight?","Did you watch the match?","Send me the location please"], s:["Good morning 🌞","Done with work?","Thanks for the help"], m:["God keep you safe, my son","Call us when you are free","Did you eat?"] },
    names:{ a:"Ahmed", s:"Sara", m:"Mom" }, welcome:"Hello! This is a demo message 👋",
    system:`You are smart glasses worn by the user. You see what they see through the camera and analyze anything in front of them. Reply with JSON only, no explanation or fences, in this shape:
{"title":"","summary":"","text":"","objects":[{"label":"","note":"","box":[x,y,w,h]}]}
box values are fractions 0 to 1 of the image size, (x,y) is the top-left corner. List at most 6 important things. label is one or two words. note is up to 9 words giving the most useful fact: what it is, what it does, a warning, a rough price, calories for food, plant name, the solution to a problem, and so on. title up to 4 words. summary one or two sentences. text: any readable text in the image translated to English if it is in another language, otherwise empty. Write in English.`,
  },
};
const COLORS = { a:"#1E5BFF", s:"#18B67A", m:"#E5384F" };

/* =================== core state =================== */
const $ = (id) => document.getElementById(id);
const video = $("vid");
let lang = "ar", facing = "environment", mirror = false, stereo = false, voiceOn = false, paused = false, demoOn = true;
let started = false, camOk = true, stream = null, landmarker = null, handStatus = "idle", arHidden = false;
let apiKey = "", ytIds = ["jNQXAC9IVRw"]; const titles = {};
try { apiKey = localStorage.getItem("sg_key") || ""; const l = JSON.parse(localStorage.getItem("sg_yt") || "null"); if (Array.isArray(l) && l.length) ytIds = l; } catch (e) {}
const S = () => STR[lang], rtl = () => lang === "ar";
let W = innerWidth, H = innerHeight, dpr = Math.min(devicePixelRatio || 1, 2);
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
function h(tag, cls, text, data) { const n = document.createElement(tag); if (cls) n.className = cls; if (text != null) n.textContent = text; if (data) Object.entries(data).forEach(([k, v]) => (n.dataset[k] = v)); return n; }

/* =================== eyes =================== */
const E = [0, 1].map((i) => {
  const root = $("eye" + i), cv = root.querySelector("canvas");
  return { i, root, cv, cx: cv.getContext("2d"), ar: root.querySelector(".ar"), els: {}, ew: 0, eh: 0, dock: null, toasts: null, curs: [], ring: null };
});
const active = () => (stereo ? [E[0], E[1]] : [E[0]]);
function chrome(e) {
  e.dock = h("div", "dock"); e.toasts = h("div", "toasts"); e.ring = h("div", "ring");
  e.ar.append(e.toasts, e.dock, e.ring);
  for (let k = 0; k < 4; k++) { const c = h("div", "cur"); e.curs.push(c); e.ar.appendChild(c); }
}
E.forEach(chrome);
const dims = () => ({ ew: stereo ? W / 2 : W, eh: H });
function layoutEyes() {
  W = innerWidth; H = innerHeight; dpr = Math.min(devicePixelRatio || 1, 2);
  const { ew, eh } = dims(); E[1].root.hidden = !stereo;
  active().forEach((e) => {
    e.ew = ew; e.eh = eh; e.cv.width = ew * dpr; e.cv.height = eh * dpr;
    const fs = clamp(ew / 42, 12, 20); e.dock.style.fontSize = fs + "px"; e.toasts.style.fontSize = fs + "px";
    e.ar.dir = rtl() ? "rtl" : "ltr";
  });
  wins.forEach(geo);
}
addEventListener("resize", layoutEyes);

/* =================== windows model =================== */
let wins = [], zTop = 10, nextId = 1;
const WDEF = { youtube: 0.46, chat: 0.32, analysis: 0.3, clock: 0.2 };
const getWin = (id) => wins.find((w) => w.id === id);
function addWin(type) {
  const ex = wins.find((w) => w.type === type);
  if (ex && type !== "youtube") { ex.z = ++zTop; geo(ex); return ex; }
  const n = wins.length, w = WDEF[type];
  const win = { id: nextId++, type, w, x: clamp(0.06 + ((n * 0.07) % 0.4), 0, 0.98 - w), y: 0.14 + ((n * 0.06) % 0.3), z: ++zTop, vid: null, cid: "a", pl: [null, null] };
  wins.push(win); active().forEach((e) => ensureEl(win, e)); return win;
}
function removeWin(id) {
  const w = getWin(id); if (!w) return;
  E.forEach((e) => { try { w.pl[e.i]?.destroy?.(); } catch (_) {} w.pl[e.i] = null; e.els[id]?.remove(); delete e.els[id]; });
  wins = wins.filter((x) => x.id !== id);
}
function ensureEl(win, e) {
  if (e.els[win.id]) return;
  const el = h("div", "win", null, { id: win.id });
  const hd = h("div", "hd"), bt = h("div", "bt");
  hd.append(h("span", "tt"));
  [["small", "−"], ["big", "+"], ["close", "✕"]].forEach(([a, t]) => bt.appendChild(h("button", null, t, { act: a })));
  hd.appendChild(bt); el.append(hd, h("div", "bd"), h("div", "rz"));
  e.ar.insertBefore(el, e.toasts); e.els[win.id] = el; paint(win, e); geo(win);
}
function geo(win) {
  active().forEach((e) => {
    const el = e.els[win.id]; if (!el) return;
    el.style.left = win.x * 100 + "%"; el.style.top = win.y * 100 + "%"; el.style.width = win.w * 100 + "%"; el.style.zIndex = win.z;
    el.style.fontSize = clamp((win.w * e.ew) / 19, 11, 26) + "px";
  });
}

/* =================== window content =================== */
function paint(win, e) {
  const el = e.els[win.id]; if (!el) return;
  el.dir = rtl() ? "rtl" : "ltr"; el.querySelector(".tt").textContent = S().wt[win.type];
  const bd = el.querySelector(".bd");
  if (win.type === "youtube") return paintYT(win, e, el, bd);
  bd.innerHTML = "";
  if (win.type === "chat") paintChat(win, bd);
  else if (win.type === "analysis") paintAn(bd);
  else paintClock(bd);
}
const paintAll = (type) => wins.forEach((w) => { if (!type || w.type === type) active().forEach((e) => paint(w, e)); });

function paintChat(win, bd) {
  const t = S(); unread[win.cid] = 0;
  const tabs = h("div", "tabs");
  Object.keys(t.names).forEach((id) => {
    const tab = h("div", "tab" + (id === win.cid ? " on" : ""), t.names[id], { act: "chat:open:" + id });
    if (unread[id]) tab.appendChild(h("i", null, String(unread[id])));
    tabs.appendChild(tab);
  });
  const th = h("div", "thread"), msgs = threads[win.cid].slice(-5);
  if (!msgs.length) th.appendChild(h("div", "muted", t.noMsg));
  msgs.forEach((m) => th.appendChild(h("div", "m " + (m.me ? "me" : "them"), m.text)));
  const rep = h("div", "rep");
  t.reply.forEach((r, i) => rep.appendChild(h("button", null, r, { act: "chat:reply:" + i })));
  rep.appendChild(h("button", null, "🎤", { act: "chat:mic" }));
  bd.append(tabs, th, rep);
}
function paintAn(bd) {
  if (!scene.title) { bd.appendChild(h("div", "muted", S().noAn)); return; }
  bd.appendChild(h("div", null, scene.title)).style.fontWeight = "800";
  bd.appendChild(h("div", "muted", scene.summary));
  objs.forEach((o) => { const it = h("div", "it"); it.append(h("b", null, o.label + ": "), h("span", null, o.note)); bd.appendChild(it); });
  if (scene.text) bd.appendChild(h("div", "muted", S().text + ": " + scene.text));
}
function paintClock(bd) {
  const d = new Date(), loc = lang === "ar" ? "ar-EG" : "en-US", c = h("div", "clk");
  c.append(h("div", "big", d.toLocaleTimeString("en-GB")), h("div", "muted", d.toLocaleDateString(loc, { weekday: "long", day: "numeric", month: "long" })));
  bd.appendChild(c);
}
setInterval(() => paintAll("clock"), 1000);

/* ---- YouTube ---- */
const parseYT = (s) => { const m = String(s).trim().match(/(?:youtu\.be\/|v=|embed\/|shorts\/)([\w-]{11})/) || String(s).trim().match(/^([\w-]{11})$/); return m ? m[1] : null; };
async function fetchTitle(id) {
  if (titles[id]) return;
  titles[id] = "…";
  try { const r = await fetch("https://www.youtube.com/oembed?format=json&url=https://www.youtube.com/watch?v=" + id); const j = await r.json(); titles[id] = j.title || ""; } catch (e) { titles[id] = ""; }
  paintAll("youtube");
}
let ytReady = null;
function loadYT() {
  if (ytReady) return ytReady;
  ytReady = new Promise((res) => {
    if (window.YT && YT.Player) return res(true);
    window.onYouTubeIframeAPIReady = () => res(true);
    const s = document.createElement("script"); s.src = "https://www.youtube.com/iframe_api"; s.onerror = () => res(false); document.head.appendChild(s);
    setTimeout(() => res(!!(window.YT && YT.Player)), 7000);
  });
  return ytReady;
}
function paintYT(win, e, el, bd) {
  const key = (win.vid || "list") + "|" + lang + "|" + ytIds.join() + "|" + Object.values(titles).join("").length;
  if (win.vid) { if (el._vid === win.vid) return; }
  else if (el._key === key) return;
  el._vid = win.vid; el._key = key;
  try { win.pl[e.i]?.destroy?.(); } catch (_) {} win.pl[e.i] = null;
  bd.innerHTML = "";
  if (!win.vid) {
    bd.appendChild(h("div", "muted", ytIds.length ? S().ytPick : S().ytEmpty)).style.marginBottom = ".5em";
    const g = h("div", "grid");
    ytIds.slice(0, 8).forEach((id) => {
      fetchTitle(id); const v = h("div", "vid", null, { act: "yt:pick:" + id }), img = h("img"); img.src = "https://i.ytimg.com/vi/" + id + "/hqdefault.jpg"; img.alt = "";
      v.append(img, h("span", null, titles[id] || (id === "jNQXAC9IVRw" ? S().demoVid : "▶"))); g.appendChild(v);
    });
    bd.appendChild(g); return;
  }
  const pl = h("div", "pl"); pl.appendChild(h("div")); bd.appendChild(pl);
  const ctl = h("div", "ctl");
  [["yt:back", "☰"], ["yt:seek:-10", "⏪"], ["yt:toggle", "⏯"], ["yt:seek:10", "⏩"], ["yt:mute", "🔊"]].forEach(([a, t]) => ctl.appendChild(h("button", null, t, { act: a })));
  bd.appendChild(ctl); mountPlayer(win, e, el, pl.firstChild);
}
async function mountPlayer(win, e, el, holder) {
  const ok = await loadYT(); if (!el.isConnected || el._vid !== win.vid) return;
  if (ok) {
    win.pl[e.i] = new YT.Player(holder, { width: "100%", height: "100%", videoId: win.vid,
      playerVars: { autoplay: 1, playsinline: 1, controls: 0, rel: 0, modestbranding: 1, fs: 0, disablekb: 1, mute: e.i ? 1 : 0 },
      events: { onReady: (ev) => {
        if (e.i === 1) { ev.target.mute(); try { ev.target.seekTo(win.pl[0].getCurrentTime(), true); } catch (_) {} }
        ev.target.playVideo(); } } });
  } else {
    const f = h("iframe"); f.src = `https://www.youtube.com/embed/${win.vid}?autoplay=1&playsinline=1&mute=${e.i ? 1 : 0}`; f.allow = "autoplay; encrypted-media"; holder.replaceWith(f);
  }
}
setInterval(() => {
  if (!stereo) return;
  wins.forEach((w) => { const a = w.pl[0], b = w.pl[1]; if (!a || !b || !a.getCurrentTime || !b.getCurrentTime) return;
    try { const ta = a.getCurrentTime(), tb = b.getCurrentTime(); if (Math.abs(ta - tb) > 0.5) b.seekTo(ta, true);
      const sa = a.getPlayerState(), sb = b.getPlayerState(); if (sa === 1 && sb !== 1) b.playVideo(); if (sa === 2 && sb === 1) b.pauseVideo(); b.mute(); } catch (_) {} });
}, 1000);

/* =================== messages + notifications =================== */
const threads = { a: [], s: [], m: [] }, unread = { a: 0, s: 0, m: 0 };
let toastList = [], toastId = 1, ac = null;
function beep() {
  try { ac = ac || new (window.AudioContext || window.webkitAudioContext)(); const o = ac.createOscillator(), g = ac.createGain();
    o.type = "sine"; o.frequency.setValueAtTime(880, ac.currentTime); o.frequency.setValueAtTime(1175, ac.currentTime + 0.1);
    g.gain.setValueAtTime(0.0001, ac.currentTime); g.gain.exponentialRampToValueAtTime(0.25, ac.currentTime + 0.02); g.gain.exponentialRampToValueAtTime(0.0001, ac.currentTime + 0.3);
    o.connect(g); g.connect(ac.destination); o.start(); o.stop(ac.currentTime + 0.32); } catch (e) {}
}
function incoming(cid, text) {
  threads[cid].push({ me: false, text });
  showToast(cid, text); beep(); navigator.vibrate?.(60);
  if (voiceOn) speak(S().names[cid] + ": " + text);
  paintAll("chat");
}
function showToast(cid, text) {
  const t = { id: toastId++, cid, text }; toastList = [...toastList.slice(-2), t]; renderToasts();
  setTimeout(() => { toastList = toastList.filter((x) => x.id !== t.id); renderToasts(); }, 7000);
}
function renderToasts() {
  active().forEach((e) => {
    e.toasts.innerHTML = ""; e.toasts.dir = rtl() ? "rtl" : "ltr";
    toastList.forEach((t) => {
      const el = h("div", "toast", null, { act: "toast:open:" + t.cid }), av = h("div", "av", S().names[t.cid][0]); av.style.background = COLORS[t.cid];
      const tx = h("div", "tx"); tx.append(h("b", null, S().names[t.cid]), h("span", null, t.text));
      el.append(av, tx, h("div", "pb")); e.toasts.appendChild(el);
    });
  });
}
function randomIncoming() { const ids = Object.keys(S().names), cid = ids[Math.floor(Math.random() * ids.length)], pool = S().msgs[cid]; incoming(cid, pool[Math.floor(Math.random() * pool.length)]); }
setInterval(() => { if (started && demoOn && !arHidden) randomIncoming(); }, 28000);
function sendReply(cid, text) { threads[cid].push({ me: true, text }); paintAll("chat"); }
function micReply(win) {
  const SR = window.SpeechRecognition || window.webkitSpeechRecognition; if (!SR) return;
  try { const r = new SR(); r.lang = rtl() ? "ar-SA" : "en-US"; r.onresult = (ev) => { const t = ev.results[0][0].transcript; if (t) sendReply(win.cid, t); }; r.start(); } catch (e) {}
}

/* =================== camera =================== */
async function startCam() {
  stream?.getTracks().forEach((t) => t.stop());
  try { stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: facing, width: { ideal: 1280 }, height: { ideal: 720 } }, audio: false });
    video.srcObject = stream; await video.play(); mirror = facing === "user"; camOk = true; } catch (e) { camOk = false; }
}
function vToE(nx, ny, ew, eh) {
  const vw = video.videoWidth || 16, vh = video.videoHeight || 9, k = Math.max(ew / vw, eh / vh), dw = vw * k, dh = vh * k;
  return { x: (ew - dw) / 2 + (mirror ? 1 - nx : nx) * dw, y: (eh - dh) / 2 + ny * dh };
}

/* =================== hand tracking =================== */
async function loadHands() {
  handStatus = "loading";
  try {
    const base = "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.14", m = await import(base + "/vision_bundle.mjs"), fs = await m.FilesetResolver.forVisionTasks(base + "/wasm");
    const make = (d) => m.HandLandmarker.createFromOptions(fs, { baseOptions: { modelAssetPath: "https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task", delegate: d }, runningMode: "VIDEO", numHands: 2 });
    try { landmarker = await make("GPU"); } catch (e) { landmarker = await make("CPU"); }
    handStatus = "ready";
  } catch (e) { handStatus = "failed"; }
}
const d2 = (a, b) => Math.hypot(a.x - b.x, a.y - b.y);
let handPts = [], hands21 = [], touchPts = {}, lastT = -1; const smoothMap = {};
function processHands() {
  if (!landmarker || video.readyState < 2 || video.currentTime === lastT) return; lastT = video.currentTime;
  let res; try { res = landmarker.detectForVideo(video, performance.now()); } catch (e) { return; }
  const { ew, eh } = dims(); handPts = []; hands21 = [];
  (res.landmarks || []).forEach((lm, i) => {
    const size = d2(lm[0], lm[9]) || 0.1, pinch = d2(lm[4], lm[8]) / size, raw = vToE((lm[4].x + lm[8].x) / 2, (lm[4].y + lm[8].y) / 2, ew, eh), id = "h" + i, prev = smoothMap[id];
    const x = prev ? prev.x + (raw.x - prev.x) * 0.55 : raw.x, y = prev ? prev.y + (raw.y - prev.y) * 0.55 : raw.y, down = prev && prev.down ? pinch < 0.55 : pinch < 0.38;
    smoothMap[id] = { x, y, down }; let ext = 0;
    [[8, 6], [12, 10], [16, 14], [20, 18]].forEach(([t, p]) => { if (d2(lm[t], lm[0]) > d2(lm[p], lm[0]) * 1.08) ext++; });
    handPts.push({ id, x, y, down, ext, wrist: vToE(lm[0].x, lm[0].y, ew, eh) }); hands21.push(lm.map((q) => vToE(q.x, q.y, ew, eh)));
  });
  Object.keys(smoothMap).forEach((k) => { if (!handPts.find((p) => p.id === k)) delete smoothMap[k]; });
}

/* =================== interaction =================== */
const press = {}; let zoom = null, palmHold = 0, fistHold = 0, gestureLock = false, startedAt = 0, everMoved = false;
function pick(x, y) {
  const els = document.elementsFromPoint(x, y), act = els.find((n) => n.dataset && n.dataset.act), we = els.find((n) => n.classList && n.classList.contains("win")), rz = els.some((n) => n.classList && n.classList.contains("rz"));
  return { act: act ? act.dataset.act : null, win: we ? getWin(+we.dataset.id) : null, rz };
}
function interact(dt) {
  processHands();
  const pts = [...handPts, ...Object.values(touchPts)], seen = new Set(), { ew, eh } = dims(), hd = handPts.filter((p) => p.down);
  if (hd.length === 2 && !arHidden) {
    const a = hd[0], b = hd[1], d = d2(a, b);
    if (!zoom) { const w = pick((a.x + b.x) / 2, (a.y + b.y) / 2).win || pick(a.x, a.y).win || pick(b.x, b.y).win; if (w) { zoom = { id: w.id, d0: Math.max(d, 30), w0: w.w }; w.z = ++zTop; Object.keys(press).forEach((k) => delete press[k]); } }
    else { const w = getWin(zoom.id); if (w) { w.w = clamp(zoom.w0 * (d / zoom.d0), 0.14, 0.95); geo(w); everMoved = true; } }
  } else zoom = null;
  pts.forEach((p) => {
    seen.add(p.id); const st = press[p.id];
    if (p.down && !st && !zoom && !arHidden) {
      const k = pick(p.x, p.y), s = { t0: performance.now(), x0: p.x, y0: p.y, moved: false, act: k.act, win: k.win, mode: null };
      if (k.win) k.win.z = ++zTop;
      if (k.act) s.mode = "click"; else if (k.win && k.rz) s.mode = "resize"; else if (k.win) { s.mode = "drag"; s.dx = k.win.x * ew - p.x; s.dy = k.win.y * eh - p.y; }
      if (k.win) geo(k.win); press[p.id] = s;
    } else if (p.down && st && !zoom) {
      if (!st.moved && Math.hypot(p.x - st.x0, p.y - st.y0) > 18) st.moved = true;
      if (st.win && st.mode === "drag" && st.moved) { st.win.x = clamp((p.x + st.dx) / ew, -st.win.w * 0.6, 1 - st.win.w * 0.4); st.win.y = clamp((p.y + st.dy) / eh, 0, 0.9); geo(st.win); everMoved = true; }
      if (st.win && st.mode === "resize") { st.win.w = clamp((p.x - st.win.x * ew) / ew, 0.14, 0.95); geo(st.win); everMoved = true; }
    } else if (!p.down && st) {
      if (st.mode === "click" && !st.moved && performance.now() - st.t0 < 1500) doAction(st.act, st.win);
      delete press[p.id];
    }
  });
  Object.keys(press).forEach((k) => { if (!seen.has(k)) delete press[k]; });
  const h0 = handPts[0];
  if (h0 && !h0.down) {
    palmHold = h0.ext === 4 ? palmHold + dt : 0; fistHold = h0.ext === 0 ? fistHold + dt : 0;
    if (!gestureLock && palmHold > 1.2) { gestureLock = true; analyze(); }
    if (!gestureLock && fistHold > 1.2) { gestureLock = true; arHidden = !arHidden; active().forEach((e) => e.ar.classList.toggle("ghost", arHidden)); }
  } else { palmHold = 0; fistHold = 0; }
  if (!h0 || (h0.ext !== 4 && h0.ext !== 0)) gestureLock = false;
}
function doAction(act, win) {
  const [a, b, c] = act.split(":");
  if (a === "add") return addWin(b);
  if (a === "stereo") return setStereo(!stereo);
  if (a === "toast") { toastList = []; renderToasts(); const w = addWin("chat"); w.cid = b; paintAll("chat"); return; }
  if (!win) return;
  if (a === "close") return removeWin(win.id);
  if (a === "big") { win.w = Math.min(0.95, win.w * 1.22); return geo(win); }
  if (a === "small") { win.w = Math.max(0.14, win.w / 1.22); return geo(win); }
  if (a === "chat") {
    if (b === "open") { win.cid = c; return paintAll("chat"); }
    if (b === "reply") return sendReply(win.cid, S().reply[+c]);
    if (b === "mic") return micReply(win);
  }
  if (a === "yt") {
    const p0 = win.pl[0];
    if (b === "pick") { win.vid = c; return active().forEach((e) => paint(win, e)); }
    if (b === "back") { win.vid = null; return active().forEach((e) => paint(win, e)); }
    if (!p0 || !p0.getPlayerState) return;
    try {
      if (b === "toggle") { const st = p0.getPlayerState(); [0, 1].forEach((i) => { const p = win.pl[i]; if (p) (st === 1 ? p.pauseVideo() : p.playVideo()); }); }
      if (b === "seek") { const t = Math.max(0, p0.getCurrentTime() + Number(c)); [0, 1].forEach((i) => win.pl[i]?.seekTo(t, true)); }
      if (b === "mute") { p0.isMuted() ? p0.unMute() : p0.mute(); }
    } catch (_) {}
  }
}

/* touch / mouse fallback */
const eyesEl = $("eyes");
const toEye = (ev) => ({ x: stereo ? ev.clientX % (W / 2) : ev.clientX, y: ev.clientY });
eyesEl.addEventListener("pointerdown", (ev) => { eyesEl.setPointerCapture(ev.pointerId); touchPts[ev.pointerId] = { id: "t" + ev.pointerId, ...toEye(ev), down: true }; });
eyesEl.addEventListener("pointermove", (ev) => { const p = touchPts[ev.pointerId]; if (p) Object.assign(p, toEye(ev)); });
["pointerup", "pointercancel"].forEach((t) => eyesEl.addEventListener(t, (ev) => { const p = touchPts[ev.pointerId]; if (p) p.down = false; setTimeout(() => delete touchPts[ev.pointerId], 60); }));

/* =================== analysis =================== */
let scene = { title: "", summary: "", text: "" }, objs = [], status = "idle", busy = false, lastAnalysis = 0, lastSig = null, lastSpoken = "";
const tiny = document.createElement("canvas"); tiny.width = 24; tiny.height = 24;
function signature() { if (video.readyState < 2) return null; const c = tiny.getContext("2d"); c.drawImage(video, 0, 0, 24, 24); const d = c.getImageData(0, 0, 24, 24).data, o = new Uint8Array(576); for (let i = 0; i < 576; i++) o[i] = (d[i * 4] + d[i * 4 + 1] + d[i * 4 + 2]) / 3; return o; }
const meanDiff = (a, b) => { let s = 0; for (let i = 0; i < a.length; i++) s += Math.abs(a[i] - b[i]); return s / a.length; };
function grabFrame() { const w = video.videoWidth, hh = video.videoHeight; if (!w || !hh) return null; const k = Math.min(1, 1024 / Math.max(w, hh)), c = document.createElement("canvas"); c.width = Math.round(w * k); c.height = Math.round(hh * k); c.getContext("2d").drawImage(video, 0, 0, c.width, c.height); return c.toDataURL("image/jpeg", 0.8).split(",")[1]; }
const num = (v) => { v = Number(v); return isFinite(v) ? clamp(v, 0, 1) : null; };
async function analyze() {
  if (busy || !camOk) return; const frame = grabFrame(); if (!frame) return; busy = true; status = "analyzing"; const sig = signature();
  try {
    const headers = { "Content-Type": "application/json" };
    if (apiKey) { headers["x-api-key"] = apiKey; headers["anthropic-version"] = "2023-06-01"; headers["anthropic-dangerous-direct-browser-access"] = "true"; }
    const res = await fetch("https://api.anthropic.com/v1/messages", { method: "POST", headers, body: JSON.stringify({ model: "claude-sonnet-4-6", max_tokens: 1000, system: S().system,
      messages: [{ role: "user", content: [{ type: "image", source: { type: "base64", media_type: "image/jpeg", data: frame } }, { type: "text", text: rtl() ? "حلل ما أراه الآن." : "Analyze what I see now." }] }] }) });
    const j = await res.json(); if (j.error) { const er = new Error("api"); er.auth = res.status === 401; throw er; }
    const raw = (j.content || []).map((b) => b.text || "").join(""), o = JSON.parse(raw.slice(raw.indexOf("{"), raw.lastIndexOf("}") + 1));
    objs = (o.objects || []).slice(0, 6).map((ob) => { const b = Array.isArray(ob.box) ? ob.box.map(num) : []; return b.length < 4 || b.some((v) => v === null) ? null : { label: String(ob.label || ""), note: String(ob.note || ""), b }; }).filter(Boolean);
    scene = { title: String(o.title || ""), summary: String(o.summary || ""), text: String(o.text || "") }; status = "live"; paintAll("analysis");
    if (voiceOn && scene.title && scene.title !== lastSpoken) { lastSpoken = scene.title; speak(scene.title + ". " + scene.summary); }
  } catch (e) { status = !apiKey || e.auth ? "errKey" : "errNet"; }
  finally { busy = false; lastAnalysis = performance.now(); lastSig = sig; }
}
setInterval(() => { if (!started || paused || busy || !camOk) return; const since = performance.now() - lastAnalysis, sig = signature(); if (!sig) return;
  const diff = lastSig ? meanDiff(sig, lastSig) : 999; if (since > 12000 || (since > 3500 && diff > 14)) analyze(); }, 800);
function speak(text) { if (!window.speechSynthesis) return; speechSynthesis.cancel(); const u = new SpeechSynthesisUtterance(text); u.lang = rtl() ? "ar-SA" : "en-US"; const v = speechSynthesis.getVoices().find((x) => x.lang.toLowerCase().startsWith(lang)); if (v) u.voice = v; speechSynthesis.speak(u); }

/* =================== drawing (camera layer) =================== */
const LINES = [[0,1],[1,2],[2,3],[3,4],[0,5],[5,6],[6,7],[7,8],[5,9],[9,10],[10,11],[11,12],[9,13],[13,14],[14,15],[15,16],[13,17],[17,18],[18,19],[19,20],[0,17]];
const font = (n, w = 500) => `${w} ${Math.round(n)}px "Segoe UI",Tahoma,system-ui,sans-serif`;
function rr(c, x, y, w, hh, r) { c.beginPath(); c.moveTo(x + r, y); c.arcTo(x + w, y, x + w, y + hh, r); c.arcTo(x + w, y + hh, x, y + hh, r); c.arcTo(x, y + hh, x, y, r); c.arcTo(x, y, x + w, y, r); c.closePath(); }
function drawCam(e) {
  const c = e.cx, { ew, eh } = e, R = rtl(), s = clamp(Math.min(ew, eh * 1.1) / 430, 0.72, 1.3);
  c.setTransform(dpr, 0, 0, dpr, 0, 0); c.clearRect(0, 0, ew, eh); c.direction = R ? "rtl" : "ltr"; c.textAlign = "start";
  if (camOk && video.readyState >= 2) {
    const vw = video.videoWidth, vh = video.videoHeight, k = Math.max(ew / vw, eh / vh), dw = vw * k, dh = vh * k, ox = (ew - dw) / 2, oy = (eh - dh) / 2;
    if (mirror) { c.save(); c.translate(ew, 0); c.scale(-1, 1); c.drawImage(video, ox, oy, dw, dh); c.restore(); } else c.drawImage(video, ox, oy, dw, dh);
  } else { c.fillStyle = "#0B1B4D"; c.fillRect(0, 0, ew, eh); if (started && !camOk) { c.fillStyle = "#BFD4FF"; c.font = font(15 * s); c.textAlign = "center"; c.fillText(S().noCam, ew / 2, eh / 2); c.textAlign = "start"; } }
  const g = c.createRadialGradient(ew / 2, eh / 2, Math.min(ew, eh) * 0.35, ew / 2, eh / 2, Math.max(ew, eh) * 0.8); g.addColorStop(0, "rgba(5,11,36,0)"); g.addColorStop(1, "rgba(5,11,36,.55)"); c.fillStyle = g; c.fillRect(0, 0, ew, eh);
  if (!started) return;
  objs.forEach((o) => { // object brackets + small label
    const p1 = vToE(o.b[0], o.b[1], ew, eh), p2 = vToE(o.b[0] + o.b[2], o.b[1] + o.b[3], ew, eh), x = Math.min(p1.x, p2.x), y = Math.min(p1.y, p2.y), w = Math.abs(p2.x - p1.x), hh = Math.abs(p2.y - p1.y), L = Math.min(16 * s, w / 3, hh / 3);
    c.strokeStyle = "rgba(191,212,255,.9)"; c.lineWidth = 2; c.beginPath();
    [[x, y, 1, 1], [x + w, y, -1, 1], [x, y + hh, 1, -1], [x + w, y + hh, -1, -1]].forEach(([px, py, dx, dy]) => { c.moveTo(px + dx * L, py); c.lineTo(px, py); c.lineTo(px, py + dy * L); }); c.stroke();
    c.font = font(12 * s, 600); const tw = c.measureText(o.label).width + 14 * s, lx = clamp(x, 4, ew - tw - 4), ly = Math.max(4, y - 22 * s);
    rr(c, lx, ly, tw, 20 * s, 10 * s); c.fillStyle = "rgba(30,91,255,.88)"; c.fill(); c.fillStyle = "#fff"; c.fillText(o.label, R ? lx + tw - 7 * s : lx + 7 * s, ly + 14 * s);
  });
  hands21.forEach((pts) => { c.strokeStyle = "rgba(191,212,255,.55)"; c.lineWidth = 2; c.beginPath(); LINES.forEach(([a, b]) => { c.moveTo(pts[a].x, pts[a].y); c.lineTo(pts[b].x, pts[b].y); }); c.stroke(); c.fillStyle = "#fff"; pts.forEach((p) => { c.beginPath(); c.arc(p.x, p.y, 2.4, 0, 7); c.fill(); }); });
  const label = paused ? S().paused : status === "analyzing" ? S().analyzing : status === "live" ? S().live : status === "errKey" ? S().errKey : status === "errNet" ? S().errNet : "";
  const hs = handStatus === "loading" ? S().handsLoading : handStatus === "failed" ? S().handsFailed : handStatus === "ready" ? S().handsReady + (handPts.length ? " ✋" + handPts.length : "") : "";
  const sx = R ? ew - 12 : 12, top = (stereo ? 20 : 52) * s; c.font = font(12 * s, 600);
  c.fillStyle = status.startsWith("err") ? "#FF4D6A" : status === "analyzing" ? "#FFC857" : "#3DDC97"; c.beginPath(); c.arc(R ? sx - 4 : sx + 4, top - 4, 4, 0, 7); c.fill();
  c.fillStyle = "#fff"; c.fillText(label, R ? sx - 14 : sx + 14, top); c.fillStyle = "#BFD4FF"; c.font = font(11 * s); c.fillText(hs, sx, top + 16 * s);
  if (!everMoved && (performance.now() - startedAt) / 1000 < 30 && wins.length) {
    c.textAlign = "center"; c.font = font(12.5 * s, 600); const t = S().hint, tw = Math.min(ew - 16, c.measureText(t).width + 26); rr(c, (ew - tw) / 2, eh - 78 * s, tw, 28 * s, 14 * s); c.fillStyle = "rgba(30,91,255,.85)"; c.fill(); c.fillStyle = "#fff"; c.fillText(t, ew / 2, eh - 59 * s); c.textAlign = "start";
  }
}
function updateCursors() {
  const pts = [...handPts, ...Object.values(touchPts)], h0 = handPts[0], prog = Math.min(1, (palmHold || fistHold) / 1.2);
  active().forEach((e) => {
    e.curs.forEach((c, k) => { const p = pts[k]; if (!p) { c.style.display = "none"; return; } c.style.display = "block"; c.style.transform = `translate(${p.x}px,${p.y}px)`; c.classList.toggle("down", p.down); });
    if (h0 && (palmHold > 0.15 || fistHold > 0.15)) { e.ring.style.display = "block"; e.ring.style.transform = `translate(${h0.wrist.x}px,${h0.wrist.y}px)`; e.ring.style.background = `conic-gradient(${palmHold ? "#3DDC97" : "#FF4D6A"} ${prog * 360}deg, rgba(255,255,255,.18) 0)`; }
    else e.ring.style.display = "none";
  });
}
let lastFrame = performance.now();
function frame(now) { const dt = Math.min(0.1, (now - lastFrame) / 1000); lastFrame = now; if (started) interact(dt); active().forEach(drawCam); updateCursors(); requestAnimationFrame(frame); }

/* =================== UI =================== */
function setStereo(v) {
  stereo = v; $("bar").classList.toggle("hide", v);
  if (v) { layoutEyes(); wins.forEach((w) => ensureEl(w, E[1])); }
  else { wins.forEach((w) => { try { w.pl[1]?.destroy?.(); } catch (_) {} w.pl[1] = null; }); Object.keys(E[1].els).forEach((k) => E[1].els[k].remove()); E[1].els = {}; E[1].toasts.innerHTML = ""; layoutEyes(); }
  renderToasts(); refreshUI(); wins.forEach(geo);
}
function buildDock() {
  active().forEach((e) => {
    e.dock.innerHTML = ""; e.dock.dir = rtl() ? "rtl" : "ltr";
    [["add:youtube", "▶", "youtube"], ["add:chat", "💬", "chat"], ["add:analysis", "🧠", "analysis"], ["add:clock", "🕐", "clock"], ["stereo", "👓", "stereo"]].forEach(([a, ic, k]) => {
      const b = h("button", k === "stereo" && stereo ? "on" : "", null, { act: a }); b.append(h("b", null, ic), h("span", null, S().dock[k])); e.dock.appendChild(b);
    });
  });
}
function refreshUI() {
  const t = S(); document.documentElement.lang = lang; document.documentElement.dir = rtl() ? "rtl" : "ltr";
  $("sTitle").textContent = t.title; $("sSub").textContent = t.sub; $("go").textContent = t.go; $("sLang").textContent = t.lang;
  $("sSteps").innerHTML = ""; t.steps.forEach((x) => $("sSteps").appendChild(h("div", null, x)));
  $("bStereo").textContent = stereo ? t.mono : t.stereo; $("bStereo").classList.toggle("on", stereo); $("bNow").textContent = t.now;
  $("bVoice").textContent = voiceOn ? t.voiceOn : t.voiceOff; $("bVoice").classList.toggle("on", voiceOn);
  $("bPause").textContent = paused ? t.resume : t.pause; $("bPause").classList.toggle("on", paused);
  $("bDemo").textContent = demoOn ? t.demoOn : t.demoOff; $("bDemo").classList.toggle("on", demoOn); $("bMsg").textContent = t.msg;
  $("bFlip").textContent = t.flip; $("bLang").textContent = t.lang; $("bSet").textContent = t.settings;
  $("kTitle").textContent = t.settings; $("kNote").textContent = t.kNote; $("kl1").textContent = t.kl1; $("kl2").textContent = t.kl2; $("keySave").textContent = t.save;
  buildDock(); renderToasts(); wins.forEach((w) => { active().forEach((e) => { const el = e.els[w.id]; if (el) { delete el._key; paint(w, e); } }); });
  active().forEach((e) => (e.ar.dir = rtl() ? "rtl" : "ltr"));
}
$("go").onclick = async () => {
  $("start").hidden = true; started = true; startedAt = performance.now(); beep();
  try { await document.documentElement.requestFullscreen?.(); } catch (e) {}
  try { await screen.orientation?.lock?.("landscape"); } catch (e) {}
  try { await navigator.wakeLock?.request("screen"); } catch (e) {}
  layoutEyes(); addWin("analysis"); const an = wins[0]; an.x = 0.03; an.y = 0.16; geo(an);
  await startCam(); loadHands(); setTimeout(() => incoming("a", S().welcome), 3500);
};
$("sLang").onclick = $("bLang").onclick = () => { lang = lang === "ar" ? "en" : "ar"; scene = { title: "", summary: "", text: "" }; objs = []; lastAnalysis = 0; lastSig = null; refreshUI(); };
$("bStereo").onclick = () => setStereo(!stereo);
$("bNow").onclick = analyze;
$("bVoice").onclick = () => { voiceOn = !voiceOn; if (voiceOn) speak(rtl() ? "تم" : "Okay"); refreshUI(); };
$("bPause").onclick = () => { paused = !paused; refreshUI(); };
$("bDemo").onclick = () => { demoOn = !demoOn; refreshUI(); };
$("bMsg").onclick = randomIncoming;
$("bFlip").onclick = async () => { facing = facing === "environment" ? "user" : "environment"; await startCam(); };
$("bSet").onclick = () => { $("keyIn").value = apiKey; $("linksIn").value = ytIds.map((i) => "https://www.youtube.com/watch?v=" + i).join("\n"); $("settings").hidden = false; };
$("keySave").onclick = () => {
  apiKey = $("keyIn").value.trim(); const ids = $("linksIn").value.split(/\s+/).map(parseYT).filter(Boolean); if (ids.length) ytIds = [...new Set(ids)];
  try { localStorage.setItem("sg_key", apiKey); localStorage.setItem("sg_yt", JSON.stringify(ytIds)); } catch (e) {}
  $("settings").hidden = true; lastAnalysis = 0; wins.forEach((w) => { if (w.type === "youtube" && !w.vid) active().forEach((e) => { const el = e.els[w.id]; if (el) delete el._key; paint(w, e); }); });
};
layoutEyes(); refreshUI(); requestAnimationFrame(frame);
</script>
</body>
</html>
