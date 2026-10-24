<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, user-scalable=no">
<title>نظارتي AR</title>
<style>
  :root { --blue:#0064E0; --ice:#B8C2D4; --glass:rgba(24,26,31,.84); --line:rgba(255,255,255,.14); }
  * { box-sizing:border-box; margin:0; }
  html,body { height:100%; background:#050B24; color:#fff; overflow:hidden; font-family:'Segoe UI',Tahoma,system-ui,sans-serif; }
  #eyes { position:fixed; inset:0; display:flex; direction:ltr; touch-action:none; user-select:none; -webkit-user-select:none; }
  .eye { position:relative; flex:1; overflow:hidden; background:#050B24; }
  .eye[hidden] { display:none; }
  .eye + .eye { border-inline-start:3px solid #000; }
  .cam { position:absolute; inset:0; width:100%; height:100%; pointer-events:none; }
  .ar { position:absolute; inset:0; }
  .ar.ghost > .win, .ar.ghost > .toasts, .ar.ghost > .pod, .ar.ghost > .pm { opacity:0; pointer-events:none; }
  #vid { position:fixed; width:2px; height:2px; opacity:0; pointer-events:none; }

  /* windows */
  .win { position:absolute; background:var(--glass); backdrop-filter:blur(18px); -webkit-backdrop-filter:blur(18px);
    border:1px solid var(--line); border-radius:1.5em; overflow:hidden; box-shadow:0 14px 36px rgba(0,0,0,.45); }
  .win.pinned { border-color:#4C9BFF; box-shadow:0 0 0 2px rgba(76,155,255,.55),0 14px 36px rgba(0,0,0,.45); }
  .hd { display:flex; align-items:center; justify-content:space-between; gap:.5em; padding:.45em .75em .2em; cursor:grab; }
  .tt { font-weight:600; font-size:.95em; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
  .bt { display:flex; gap:.3em; direction:ltr; flex:0 0 auto; }
  .bt button, .ctl button { min-width:2.1em; height:2.1em; border:none; border-radius:50%; background:rgba(255,255,255,.14); color:#fff; font-size:1em; cursor:pointer; }
  .bt button.on { background:var(--blue); }
  .bd { padding:.35em .7em .4em; }
  .gb { width:26%; height:.32em; margin:.1em auto .6em; border-radius:1em; background:rgba(255,255,255,.5); }
  .rz { position:absolute; right:0; bottom:0; width:2.4em; height:2.4em; background:linear-gradient(135deg,transparent 52%,rgba(255,255,255,.7) 52%); border-bottom-right-radius:1.5em; }
  .pl { aspect-ratio:16/9; background:#000; border-radius:.8em; overflow:hidden; }
  .pl iframe { width:100%; height:100%; display:block; border:0; pointer-events:none; }
  .ctl { display:flex; justify-content:center; gap:.5em; margin-top:.5em; direction:ltr; }
  .grid { display:grid; grid-template-columns:1fr 1fr; gap:.5em; margin-top:.4em; }
  .vid { background:rgba(255,255,255,.08); border-radius:.9em; overflow:hidden; cursor:pointer; }
  .vid img { width:100%; aspect-ratio:16/9; object-fit:cover; display:block; pointer-events:none; }
  .vid span { display:block; padding:.3em .5em; font-size:.8em; line-height:1.4; max-height:2.9em; overflow:hidden; pointer-events:none; }
  .muted { color:var(--ice); font-size:.9em; line-height:1.7; }
  .tabs { display:flex; gap:.4em; margin-bottom:.5em; flex-wrap:wrap; }
  .tab { padding:.4em .9em; border-radius:1em; background:rgba(255,255,255,.12); font-size:.9em; position:relative; cursor:pointer; }
  .tab.on { background:var(--blue); }
  .tab i { position:absolute; top:-.2em; inset-inline-end:-.2em; background:#FF4D6A; border-radius:50%; min-width:1.3em; height:1.3em; font-style:normal; font-size:.75em; display:flex; align-items:center; justify-content:center; }
  .thread { display:flex; flex-direction:column; gap:.35em; min-height:5em; }
  .m { max-width:82%; padding:.4em .75em; border-radius:1em; line-height:1.5; font-size:.95em; }
  .m.me { align-self:flex-end; background:var(--blue); } .m.them { align-self:flex-start; background:rgba(255,255,255,.16); }
  .rep { display:flex; flex-wrap:wrap; gap:.4em; margin-top:.55em; }
  .rep button { border:1px solid var(--line); background:rgba(255,255,255,.1); color:#fff; border-radius:1em; padding:.5em 1em; font-size:.9em; cursor:pointer; }
  .it { padding:.35em 0; border-bottom:1px solid rgba(255,255,255,.1); line-height:1.5; font-size:.95em; }
  .it b { color:#fff; } .it span { color:var(--ice); }
  .clk { text-align:center; } .clk .num { font-size:2.4em; font-weight:800; letter-spacing:.03em; direction:ltr; }
  .lib { display:grid; grid-template-columns:repeat(3,1fr); gap:.5em; }
  .tile { display:flex; flex-direction:column; align-items:center; gap:.25em; padding:.8em .3em; border-radius:1.2em; background:rgba(255,255,255,.08); font-size:.85em; text-align:center; cursor:pointer; }
  .tile b { font-size:1.9em; font-weight:400; pointer-events:none; } .tile span { pointer-events:none; }
  .mv { font-size:2em; font-weight:800; direction:ltr; text-align:center; margin:.15em 0; }
  .srow { display:flex; gap:.4em; align-items:center; margin-bottom:.3em; }
  .fld { flex:1; min-width:0; padding:.55em .8em; border-radius:1em; background:rgba(0,0,0,.35); border:1px solid var(--line); font-size:.95em; cursor:text; overflow:hidden; white-space:nowrap; text-overflow:ellipsis; margin:.2em 0; }
  .fld span { pointer-events:none; }
  .fld.ph { color:var(--ice); } .fld.on { border-color:#4C9BFF; box-shadow:0 0 0 2px rgba(76,155,255,.4); }

  /* virtual keyboard */
  .kr { display:flex; gap:.25em; margin-bottom:.25em; direction:ltr; }
  .key { flex:1; min-width:0; height:2.5em; display:flex; align-items:center; justify-content:center; border-radius:.7em; background:rgba(255,255,255,.14); font-size:1.05em; cursor:pointer; }
  .key.w2 { flex:1.6; } .key.sp { flex:4.5; font-size:.85em; } .key.on { background:var(--blue); }

  /* hand menu, system pod, toasts, cursors */
  .pm { position:absolute; inset:0; z-index:960; pointer-events:none; display:none; }
  .pb { position:absolute; left:0; top:0; border-radius:50%; background:rgba(24,26,31,.94); border:1.5px solid rgba(255,255,255,.35); display:flex; align-items:center; justify-content:center; pointer-events:auto; cursor:pointer; box-shadow:0 6px 18px rgba(0,0,0,.5); }
  .pb b { font-weight:400; pointer-events:none; } .pb span { position:absolute; top:100%; margin-top:.15em; font-size:.3em; white-space:nowrap; pointer-events:none; text-shadow:0 1px 4px #000; }
  .dock { position:absolute; bottom:3%; left:50%; transform:translateX(-50%); display:flex; gap:.45em; padding:.5em .6em; z-index:900;
    background:rgba(24,26,31,.9); border:1px solid var(--line); border-radius:2.4em; backdrop-filter:blur(18px); -webkit-backdrop-filter:blur(18px); }
  .dock button { border:none; background:rgba(255,255,255,.08); color:#fff; border-radius:1.6em; padding:.5em .6em; font-size:1em; cursor:pointer; display:flex; flex-direction:column; align-items:center; gap:.1em; min-width:3.4em; }
  .dock button b { font-size:1.3em; font-weight:400; } .dock button span { font-size:.68em; opacity:.8; }
  .pod { position:absolute; top:2.5%; left:50%; transform:translateX(-50%); z-index:890; padding:.3em 1em; border-radius:1.4em; background:rgba(24,26,31,.82); border:1px solid var(--line); direction:ltr; white-space:nowrap; }
  .toasts { position:absolute; top:11%; left:50%; transform:translateX(-50%); width:56%; display:flex; flex-direction:column; gap:.5em; z-index:950; }
  .toast { display:flex; gap:.7em; align-items:center; padding:.7em .9em; border-radius:1.4em; background:rgba(24,26,31,.94); border:1px solid var(--line);
    box-shadow:0 8px 24px rgba(0,0,0,.45); position:relative; overflow:hidden; animation:slide .35s ease-out; }
  .toast .av { width:2.4em; height:2.4em; border-radius:50%; display:flex; align-items:center; justify-content:center; font-weight:800; flex:0 0 auto; }
  .toast .tx { min-width:0; } .toast .tx b { display:block; } .toast .tx span { display:block; color:var(--ice); font-size:.92em; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
  .toast .pb2 { position:absolute; bottom:0; inset-inline-start:0; height:3px; background:#4C9BFF; animation:shrink 7s linear forwards; }
  @keyframes slide { from { transform:translateY(-30px); opacity:0; } to { transform:none; opacity:1; } }
  @keyframes shrink { from { width:100%; } to { width:0; } }
  .cur { position:absolute; left:0; top:0; width:28px; height:28px; margin:-14px 0 0 -14px; border-radius:50%; border:2.5px solid #fff; background:rgba(255,255,255,.14);
    z-index:1000; pointer-events:none; display:none; box-shadow:0 0 14px rgba(255,255,255,.6); }
  .cur.down { background:rgba(0,100,224,.95); width:20px; height:20px; margin:-10px 0 0 -10px; }
  .ring { position:absolute; width:64px; height:64px; margin:-32px 0 0 -32px; border-radius:50%; z-index:999; pointer-events:none; display:none;
    -webkit-mask:radial-gradient(circle,transparent 56%,#000 58%); mask:radial-gradient(circle,transparent 56%,#000 58%); }

  /* start screen (only DOM screen) */
  .pill { border:1px solid var(--line); background:var(--glass); color:#fff; border-radius:20px; padding:8px 14px; font-size:13px; cursor:pointer; }
  .overlay { position:fixed; inset:0; z-index:50; display:flex; align-items:center; justify-content:center; padding:20px; overflow-y:auto; background:radial-gradient(circle at 50% 20%,#14306f,#050B24 70%); }
  .overlay[hidden] { display:none; }
  .panel { width:100%; max-width:480px; text-align:center; display:flex; flex-direction:column; gap:12px; }
  .panel h1 { font-size:27px; } .panel p { color:var(--ice); line-height:1.7; font-size:15px; }
  .steps { text-align:start; background:rgba(255,255,255,.06); border:1px solid var(--line); border-radius:18px; padding:14px 18px; display:flex; flex-direction:column; gap:8px; font-size:14px; line-height:1.65; }
  .big { border:none; background:var(--blue); color:#fff; font-size:19px; font-weight:700; padding:15px; border-radius:16px; cursor:pointer; }
  .f { width:100%; border:1px solid var(--line); background:rgba(255,255,255,.08); color:#fff; border-radius:12px; padding:12px; font-size:14px; outline:none; font-family:inherit; direction:ltr; }
  label { text-align:start; font-size:13px; color:var(--ice); }
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

<div class="overlay" id="start"><div class="panel">
  <h1 id="sTitle"></h1><p id="sSub"></p><div class="steps" id="sSteps"></div>
  <label id="kl1"></label><input class="f" id="keyIn" type="password" placeholder="sk-ant-..." autocomplete="off">
  <label id="kl2"></label><input class="f" id="ytKeyIn" type="password" placeholder="AIza..." autocomplete="off">
  <button class="big" id="go"></button><button class="pill" id="sLang" style="align-self:center"></button>
</div></div>

<script type="module">
/* =================== strings =================== */
const STR = {
  ar: {
    title:"نظارتي AR", sub:"كل شيء داخل الواقع المعزز: قائمة تظهر أمام يدك، نوافذ تلصقها في مكانها، ولوحة مفاتيح افتراضية للبحث. الذكاء الاصطناعي يرى ما تراه.",
    steps:["١. اضغط «ابدأ»، ثم ضع الجوال أفقياً في حامل رأس (Cardboard) أو أمسكه أمامك.","٢. افتح كفّك أمام الكاميرا: تظهر دائرة تطبيقات أمام يدك. اختر تطبيقاً بقرصة من يدك الأخرى (أو أنزل يدك واقرص خلال 5 ثوان).","٣. اقرص أي نافذة وحرّكها. للتكبير والتصغير: + و −، أو الزاوية السفلية، أو اقرصها بيديك الاثنتين وباعد بينهما.","٤. اضغط 📌 على نافذة لتلصقها في مكانها من الغرفة، فتبقى ثابتة عندما تدور بالجوال.","٥. اضغط على خانة البحث لتظهر لوحة المفاتيح. القبضة ثانية وربع = إخفاء أو إظهار الكل."],
    go:"ابدأ", lang:"EN", kl1:"مفتاح Anthropic API (للتحليل والشطرنج)", kl2:"مفتاح YouTube Data API (اختياري، لبحث أدق)",
    stereo:"عدسة النظارة (عينان)", mono:"شاشة عادية", voiceOn:"🔊 الصوت: يعمل", voiceOff:"🔇 الصوت: متوقف", pause:"إيقاف التحليل", resume:"استئناف التحليل",
    demoOn:"رسائل تجريبية: تعمل", demoOff:"رسائل تجريبية: متوقفة", flip:"قلب الكاميرا",
    dock:{ library:"التطبيقات", analysis:"Meta AI", youtube:"يوتيوب", chat:"الرسائل", chess:"شطرنج", clock:"الساعة", settings:"الإعدادات" },
    wt:{ youtube:"يوتيوب", chat:"الرسائل", analysis:"Meta AI", clock:"الساعة", chess:"مدرب الشطرنج", library:"التطبيقات", app:"", keyboard:"لوحة المفاتيح", settings:"الإعدادات" },
    searchPh:"ابحث في يوتيوب...", searchBtn:"بحث", recent:"الأخيرة", searching:"جاري البحث...", noRes:"لا توجد نتائج", errSearch:"تعذر البحث الآن. أعد المحاولة، أو أضف مفتاح YouTube من شاشة البداية.",
    demoVid:"فيديو تجريبي", newAppPh:"اسم التطبيق | التعليمات", addBtn:"إضافة", apps:"تطبيقاتك", sp:"مسافة", kbAr:"عربي",
    reply:["👍","تمام","بعدين أكلمك","أنا في الطريق"], noMsg:"لا رسائل بعد",
    analyzing:"يحلل...", live:"مباشر", paused:"التحليل متوقف", errKey:"أدخل مفتاح API من شاشة البداية", errNet:"تعذر الاتصال، سأعيد المحاولة", errParse:"الرد غير مفهوم، حاول مرة أخرى",
    noCam:"الكاميرا غير مفعّلة. اسمح بالإذن وافتح الصفحة عبر https.", handsLoading:"جاري تحميل تتبع اليد", handsReady:"اليد جاهزة", handsFailed:"تتبع اليد غير متاح، استخدم اللمس",
    hint:"افتح كفّك لتظهر القائمة • اقرص لتحريك النوافذ • 📌 للصق النافذة", noAn:"اسأل Meta AI عمّا أمامك، أو انتظر التحليل التلقائي...", text:"نص",
    askMic:"اسأل", now:"حلّل الآن", white:"أبيض", black:"أسود", auto:"مراقبة تلقائية", analyzeBoard:"حلّل الرقعة", chessHint:"وجّه الكاميرا على الرقعة من الأعلى واجعلها كلها داخل الصورة.",
    reading:"يقرأ الرقعة...", noBoard:"لا أرى رقعة واضحة.", best:"أفضل نقلة", danger:"انتبه", conf:"الثقة", bestIs:"أفضل نقلة",
    appsHint:"أضف تطبيقاتك من الإعدادات.", appRun:"شغّل", appSys:"أنت نظارة ذكية ترى ما يراه المستخدم. أجب بالعربية في ثلاث جمل كحد أقصى. المهمة:",
    msgs:{ a:["وينك؟ نتقابل الليلة؟","شفت المباراة؟","ابعث لي الموقع من فضلك"], s:["صباح الخير 🌞","خلصتي الشغل؟","شكراً على المساعدة"], m:["الله يحفظك يا ولدي","اتصل علينا لمن تفضى","أكلت؟"] },
    names:{ a:"أحمد", s:"سارة", m:"أمي" }, welcome:"مرحباً! هذه رسالة تجريبية 👋",
    system:`أنت نظارة ذكية يرتديها المستخدم، ترى ما يراه عبر الكاميرا وتحلل أي شيء أمامه. أعد JSON فقط بدون أي شرح أو علامات، بهذا الشكل:
{"title":"","summary":"","text":"","objects":[{"label":"","note":"","box":[x,y,w,h]}]}
box نسب من 0 إلى 1 من أبعاد الصورة، (x,y) الزاوية العليا اليسرى. اذكر حتى 6 أشياء مهمة فقط. label كلمة أو كلمتان. note حتى 9 كلمات وتعطي أعلى فائدة: ما هو، فائدته، تحذير، سعر تقريبي، سعرات للأكل، اسم النبات، حل المسألة، إلخ. title حتى 4 كلمات. summary جملة أو جملتان. text: أي نص مقروء في الصورة مترجماً للعربية إن كان بلغة أخرى، وإلا فاتركه فارغاً. اكتب بالعربية.`,
    chessAsk:"حلل الرقعة وأعطني أفضل نقلة.",
    chessSys:(side)=>`أنت مدرب شطرنج. ترى صورة لرقعة شطرنج حقيقية من الكاميرا. حدد القطع ومواضعها بدقة، وأعد JSON فقط بلا شرح أو علامات بهذا الشكل: {"board":true,"fen":"","best":"","why":"","danger":"","conf":"high|medium|low"}. best أفضل نقلة بصيغة SAN مثل Nf3. why جملتان كحد أقصى. danger أي تهديد من الخصم أو اتركه فارغاً. إن لم تكن هناك رقعة واضحة اجعل board=false واشرح السبب في why. اللاعب يلعب بالقطع ${side==="w"?"البيضاء":"السوداء"} والدور عليه. اكتب بالعربية.`,
  },
  en: {
    title:"My AR Glasses", sub:"Everything lives inside augmented reality: a menu that appears in front of your hand, windows you stick in place, and a virtual keyboard for search. The AI sees what you see.",
    steps:["1. Tap Start, then put the phone in landscape in a headset holder (Cardboard) or hold it in front of you.","2. Open your palm to the camera: an app circle appears in front of your hand. Pick an app with a pinch from your other hand (or lower your hand and pinch within 5 seconds).","3. Pinch any window and move it. To resize: + and −, the bottom corner, or pinch it with both hands and move them apart.","4. Tap 📌 on a window to stick it in place in the room, so it stays put when you turn the phone.","5. Tap the search box and the keyboard appears. Fist for 1.2s = hide or show all."],
    go:"Start", lang:"عربي", kl1:"Anthropic API key (for analysis and chess)", kl2:"YouTube Data API key (optional, for better search)",
    stereo:"Glasses lens (two eyes)", mono:"Normal screen", voiceOn:"🔊 Voice: on", voiceOff:"🔇 Voice: off", pause:"Pause analysis", resume:"Resume analysis",
    demoOn:"Demo messages: on", demoOff:"Demo messages: off", flip:"Flip camera",
    dock:{ library:"Apps", analysis:"Meta AI", youtube:"YouTube", chat:"Messages", chess:"Chess", clock:"Clock", settings:"Settings" },
    wt:{ youtube:"YouTube", chat:"Messages", analysis:"Meta AI", clock:"Clock", chess:"Chess coach", library:"Apps", app:"", keyboard:"Keyboard", settings:"Settings" },
    searchPh:"Search YouTube...", searchBtn:"Search", recent:"Recent", searching:"Searching...", noRes:"No results", errSearch:"Search failed. Try again, or add a YouTube key on the start screen.",
    demoVid:"Demo video", newAppPh:"App name | Instructions", addBtn:"Add", apps:"Your apps", sp:"space", kbAr:"عربي",
    reply:["👍","Okay","Talk later","On my way"], noMsg:"No messages yet",
    analyzing:"Analyzing...", live:"Live", paused:"Analysis paused", errKey:"Enter your API key on the start screen", errNet:"Connection failed, retrying", errParse:"Unclear reply, try again",
    noCam:"Camera is off. Allow access and open the page over https.", handsLoading:"Loading hand tracking", handsReady:"Hands ready", handsFailed:"Hand tracking unavailable, use touch",
    hint:"Open your palm for the menu • pinch to move windows • 📌 to stick a window", noAn:"Ask Meta AI about what is in front of you, or wait for auto analysis...", text:"Text",
    askMic:"Ask", now:"Analyze now", white:"White", black:"Black", auto:"Auto watch", analyzeBoard:"Analyze board", chessHint:"Point the camera at the board from above and keep the whole board in view.",
    reading:"Reading the board...", noBoard:"I can't see a clear board.", best:"Best move", danger:"Watch out", conf:"Confidence", bestIs:"Best move",
    appsHint:"Add your own apps in Settings.", appRun:"Run", appSys:"You are smart glasses that see what the user sees. Answer in English in three sentences at most. Task:",
    msgs:{ a:["Where are you? Meet tonight?","Did you watch the match?","Send me the location please"], s:["Good morning 🌞","Done with work?","Thanks for the help"], m:["God keep you safe, my son","Call us when you are free","Did you eat?"] },
    names:{ a:"Ahmed", s:"Sara", m:"Mom" }, welcome:"Hello! This is a demo message 👋",
    system:`You are smart glasses worn by the user. You see what they see through the camera and analyze anything in front of them. Reply with JSON only, no explanation or fences, in this shape:
{"title":"","summary":"","text":"","objects":[{"label":"","note":"","box":[x,y,w,h]}]}
box values are fractions 0 to 1 of the image size, (x,y) is the top-left corner. List at most 6 important things. label is one or two words. note is up to 9 words giving the most useful fact: what it is, what it does, a warning, a rough price, calories for food, plant name, the solution to a problem, and so on. title up to 4 words. summary one or two sentences. text: any readable text in the image translated to English if it is in another language, otherwise empty. Write in English.`,
    chessAsk:"Analyze the board and give me the best move.",
    chessSys:(side)=>`You are a chess coach. You see a photo of a real chess board from the camera. Identify the pieces and their squares carefully, and reply with JSON only, no explanation or fences, in this shape: {"board":true,"fen":"","best":"","why":"","danger":"","conf":"high|medium|low"}. best is the best move in SAN such as Nf3. why is two sentences at most. danger is any threat from the opponent, or empty. If there is no clear board set board=false and explain in why. The player has the ${side==="w"?"white":"black"} pieces and it is their turn. Write in English.`,
  },
};
const COLORS = { a:"#0064E0", s:"#18B67A", m:"#E5384F" };
const MENU = [["youtube","▶"],["analysis","✦"],["chess","♞"],["chat","💬"],["library","▦"],["settings","⚙"]];

/* =================== core state =================== */
const $ = (id) => document.getElementById(id);
const video = $("vid");
let lang = "ar", facing = "environment", mirror = false, stereo = false, voiceOn = false, paused = false, demoOn = true;
let started = false, camOk = true, stream = null, landmarker = null, handStatus = "idle", arHidden = false, batt = null;
let apiKey = "", ytKey = "", ytIds = ["jNQXAC9IVRw"], customApps = []; const titles = {};
let kbTarget = null, kbLayer = "ar", kbShift = false; const fields = {};
try { apiKey = localStorage.getItem("sg_key") || ""; ytKey = localStorage.getItem("sg_ytkey") || "";
  const l = JSON.parse(localStorage.getItem("sg_yt") || "null"); if (Array.isArray(l) && l.length) ytIds = l;
  const ca = JSON.parse(localStorage.getItem("sg_apps") || "[]"); if (Array.isArray(ca)) customApps = ca; } catch (e) {}
const saveLS = (k, v) => { try { localStorage.setItem(k, typeof v === "string" ? v : JSON.stringify(v)); } catch (e) {} };
const S = () => STR[lang], rtl = () => lang === "ar";
let W = innerWidth, H = innerHeight, dpr = Math.min(devicePixelRatio || 1, 2);
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
function h(tag, cls, text, data) { const n = document.createElement(tag); if (cls) n.className = cls; if (text != null) n.textContent = text; if (data) Object.entries(data).forEach(([k, v]) => (n.dataset[k] = v)); return n; }

/* =================== eyes =================== */
const E = [0, 1].map((i) => {
  const root = $("eye" + i), cv = root.querySelector("canvas");
  return { i, root, cv, cx: cv.getContext("2d"), ar: root.querySelector(".ar"), els: {}, ew: 0, eh: 0, dock: null, toasts: null, pod: null, pm: null, curs: [], ring: null };
});
const active = () => (stereo ? [E[0], E[1]] : [E[0]]);
function chrome(e) {
  e.dock = h("div", "dock"); e.toasts = h("div", "toasts"); e.ring = h("div", "ring"); e.pod = h("div", "pod"); e.pm = h("div", "pm");
  MENU.forEach(([k, ic]) => { const b = h("div", "pb", null, { act: "add:" + k }); b.append(h("b", null, ic), h("span", null, "")); e.pm.appendChild(b); });
  e.ar.append(e.toasts, e.pod, e.dock, e.pm, e.ring);
  for (let k = 0; k < 4; k++) { const c = h("div", "cur"); e.curs.push(c); e.ar.appendChild(c); }
}
E.forEach(chrome);
const dims = () => ({ ew: stereo ? W / 2 : W, eh: H });
function layoutEyes() {
  W = innerWidth; H = innerHeight; dpr = Math.min(devicePixelRatio || 1, 2);
  const { ew, eh } = dims(); E[1].root.hidden = !stereo;
  active().forEach((e) => {
    e.ew = ew; e.eh = eh; e.cv.width = ew * dpr; e.cv.height = eh * dpr;
    const fs = clamp(ew / 42, 12, 20); e.dock.style.fontSize = fs + "px"; e.toasts.style.fontSize = fs + "px"; e.pod.style.fontSize = fs + "px";
    e.ar.dir = rtl() ? "rtl" : "ltr";
  });
  wins.forEach(geo);
}
addEventListener("resize", layoutEyes);

/* =================== windows model =================== */
let wins = [], zTop = 10, nextId = 1;
const WDEF = { youtube: 0.48, chat: 0.32, analysis: 0.32, clock: 0.2, chess: 0.34, library: 0.42, app: 0.3, keyboard: 0.8, settings: 0.4 };
const getWin = (id) => wins.find((w) => w.id === id);
const rp = (w) => active().forEach((e) => paint(w, e));
function addWin(type, extra) {
  const ex = wins.find((w) => w.type === type && (type !== "app" || w.app === extra));
  if (ex && type !== "youtube") { ex.z = ++zTop; geo(ex); return ex; }
  const n = wins.length, w = WDEF[type];
  const win = { id: nextId++, type, w, x: clamp(0.06 + ((n * 0.07) % 0.4), 0, 0.98 - w), y: 0.16 + ((n * 0.06) % 0.3), z: ++zTop, vid: null, cid: "a", pl: [null, null],
    side: "w", res: null, busy: false, auto: false, last: 0, app: type === "app" ? extra : null, out: "", pin: false, items: [], srv: 0, sbusy: false, serr: "" };
  if (type === "library") { win.x = 0.5 - w / 2; win.y = 0.14; }
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
  [["pin", "📌"], ["small", "−"], ["big", "+"], ["close", "✕"]].forEach(([a, t]) => bt.appendChild(h("button", a === "pin" && win.pin ? "on" : "", t, { act: a })));
  hd.appendChild(bt); el.append(hd, h("div", "bd"), h("div", "gb"), h("div", "rz"));
  e.ar.insertBefore(el, e.toasts); e.els[win.id] = el; paint(win, e); geo(win);
}
function geo(win) {
  active().forEach((e) => {
    const el = e.els[win.id]; if (!el) return;
    el.style.left = win.x * 100 + "%"; el.style.top = win.y * 100 + "%"; el.style.width = win.w * 100 + "%"; el.style.zIndex = win.z;
    el.style.fontSize = clamp((win.w * e.ew) / 19, 11, 26) + "px";
    el.classList.toggle("pinned", !!win.pin); el.querySelector('[data-act="pin"]')?.classList.toggle("on", !!win.pin);
  });
}

/* =================== fields + virtual keyboard =================== */
function fieldEl(id, ph) { const n = h("div", "fld", null, { act: "kb:focus:" + id, fid: id, ph }); const s = h("span"); s.dir = "auto"; n.appendChild(s); return n; }
function updateFields() {
  E.forEach((e) => e.ar.querySelectorAll(".fld").forEach((n) => { const v = fields[n.dataset.fid] || ""; n.firstChild.textContent = v || n.dataset.ph; n.classList.toggle("ph", !v); n.classList.toggle("on", n.dataset.fid === kbTarget); }));
}
const KB = { en: ["qwertyuiop", "asdfghjkl", "zxcvbnm"], ar: ["ضصثقفغعهخحج", "شسيبلاتنمكط", "ئءؤرىةوزظد"], sym: ["1234567890", "-_.:/@#&?!", "=+*()%$'\"~,"] };
function paintKb(bd) {
  const t = S();
  KB[kbLayer].forEach((row, ri) => {
    const r = h("div", "kr");
    if (ri === 2 && kbLayer === "en") r.appendChild(h("div", "key w2" + (kbShift ? " on" : ""), "⇧", { act: "kb:shift" }));
    Array.from(row).forEach((ch) => { const L = kbShift && kbLayer === "en" ? ch.toUpperCase() : ch; r.appendChild(h("div", "key", L, { act: "kb:k:" + L })); });
    if (ri === 2) r.appendChild(h("div", "key w2", "⌫", { act: "kb:bs" }));
    bd.appendChild(r);
  });
  const r = h("div", "kr");
  [[t.kbAr, "kb:layer:ar", kbLayer === "ar"], ["EN", "kb:layer:en", kbLayer === "en"], ["123", "kb:layer:sym", kbLayer === "sym"]].forEach(([l, a, on]) => r.appendChild(h("div", "key w2" + (on ? " on" : ""), l, { act: a })));
  r.appendChild(h("div", "key sp", t.sp, { act: "kb:sp" }));
  r.appendChild(h("div", "key w2", "⏎", { act: "kb:go" })); r.appendChild(h("div", "key w2", "⌄", { act: "kb:close" }));
  bd.appendChild(r);
}
let ac = null;
function beep(freq = 880) {
  try { ac = ac || new (window.AudioContext || window.webkitAudioContext)(); const o = ac.createOscillator(), g = ac.createGain();
    o.type = "sine"; o.frequency.setValueAtTime(freq, ac.currentTime); if (freq === 880) o.frequency.setValueAtTime(1175, ac.currentTime + 0.1);
    g.gain.setValueAtTime(0.0001, ac.currentTime); g.gain.exponentialRampToValueAtTime(0.25, ac.currentTime + 0.02); g.gain.exponentialRampToValueAtTime(0.0001, ac.currentTime + (freq === 880 ? 0.3 : 0.07));
    o.connect(g); g.connect(ac.destination); o.start(); o.stop(ac.currentTime + (freq === 880 ? 0.32 : 0.08)); } catch (e) {}
}
function closeKb() { const k = wins.find((w) => w.type === "keyboard"); if (k) removeWin(k.id); kbTarget = null; updateFields(); }
function submitField(f) {
  if (f === "newapp") { addAppFromField(); paintAll("settings"); }
  else if (f[0] === "q") { const w = getWin(+f.slice(1)); if (w) ytSearch(w); }
  closeKb();
}
function kbAct(b, c) {
  if (b === "focus") { kbTarget = c; const n = wins.length, k = addWin("keyboard"); if (wins.length > n) { k.x = (1 - k.w) / 2; k.y = 0.5; } k.z = ++zTop; geo(k); updateFields(); return; }
  if (b === "layer") { kbLayer = c; kbShift = false; return paintAll("keyboard"); }
  if (b === "shift") { kbShift = !kbShift; return paintAll("keyboard"); }
  if (b === "close") return closeKb();
  const f = kbTarget; if (!f) return; let v = fields[f] || "";
  if (b === "k") { v += c; if (kbShift) { kbShift = false; paintAll("keyboard"); } }
  else if (b === "bs") v = Array.from(v).slice(0, -1).join("");
  else if (b === "sp") v += " ";
  else if (b === "go") return submitField(f);
  fields[f] = v; updateFields(); beep(1500);
}
function addAppFromField() {
  const l = fields.newapp || "", k = l.indexOf("|"); fields.newapp = "";
  if (k < 1) return; const n = l.slice(0, k).trim(), p = l.slice(k + 1).trim(); if (!n || !p) return;
  customApps.push({ id: "u" + Date.now(), n, p }); saveLS("sg_apps", customApps); paintAll("library");
}

/* =================== window content =================== */
function paint(win, e) {
  const el = e.els[win.id]; if (!el) return;
  el.dir = rtl() ? "rtl" : "ltr";
  el.querySelector(".tt").textContent = win.type === "app" ? (customApps.find((a) => a.id === win.app)?.n || "") : S().wt[win.type];
  const bd = el.querySelector(".bd");
  if (win.type === "youtube") return paintYT(win, e, el, bd);
  bd.innerHTML = "";
  if (win.type === "chat") paintChat(win, bd);
  else if (win.type === "analysis") paintAn(bd);
  else if (win.type === "chess") paintChess(win, bd);
  else if (win.type === "library") paintLib(bd);
  else if (win.type === "app") paintApp(win, bd);
  else if (win.type === "keyboard") paintKb(bd);
  else if (win.type === "settings") { paintSet(bd); updateFields(); }
  else paintClock(bd);
}
const paintAll = (type) => wins.forEach((w) => { if (!type || w.type === type) active().forEach((e) => paint(w, e)); });

function paintChat(win, bd) {
  const t = S(); unread[win.cid] = 0;
  const tabs = h("div", "tabs");
  Object.keys(t.names).forEach((id) => tabs.appendChild(h("div", "tab" + (id === win.cid ? " on" : ""), t.names[id], { act: "chat:open:" + id })));
  const th = h("div", "thread"), msgs = threads[win.cid].slice(-5);
  if (!msgs.length) th.appendChild(h("div", "muted", t.noMsg));
  msgs.forEach((m) => th.appendChild(h("div", "m " + (m.me ? "me" : "them"), m.text)));
  const rep = h("div", "rep");
  t.reply.forEach((r, i) => rep.appendChild(h("button", null, r, { act: "chat:reply:" + i })));
  rep.appendChild(h("button", null, "🎤", { act: "chat:mic" }));
  bd.append(tabs, th, rep);
}
function paintAn(bd) {
  if (!scene.title) bd.appendChild(h("div", "muted", S().noAn));
  else {
    bd.appendChild(h("div", null, scene.title)).style.fontWeight = "800";
    bd.appendChild(h("div", "muted", scene.summary));
    objs.forEach((o) => { const it = h("div", "it"); it.append(h("b", null, o.label + ": "), h("span", null, o.note)); bd.appendChild(it); });
    if (scene.text) bd.appendChild(h("div", "muted", S().text + ": " + scene.text));
  }
  const rep = h("div", "rep");
  rep.append(h("button", null, "🎤 " + S().askMic, { act: "an:mic" }), h("button", null, S().now, { act: "an:now" }));
  bd.appendChild(rep);
}
function paintClock(bd) {
  const d = new Date(), loc = lang === "ar" ? "ar-EG" : "en-US", c = h("div", "clk");
  c.append(h("div", "num", d.toLocaleTimeString("en-GB")), h("div", "muted", d.toLocaleDateString(loc, { weekday: "long", day: "numeric", month: "long" })));
  bd.appendChild(c);
}
function paintLib(bd) {
  const t = S(), g = h("div", "lib");
  [["youtube", "▶"], ["chat", "💬"], ["analysis", "✦"], ["chess", "♞"], ["clock", "🕐"], ["settings", "⚙"]].forEach(([k, ic]) => {
    const tl = h("div", "tile", null, { act: "add:" + k }); tl.append(h("b", null, ic), h("span", null, t.dock[k])); g.appendChild(tl);
  });
  customApps.forEach((a) => { const tl = h("div", "tile", null, { act: "app:open:" + a.id }); tl.append(h("b", null, "🔹"), h("span", null, a.n)); g.appendChild(tl); });
  bd.appendChild(g);
  if (!customApps.length) bd.appendChild(h("div", "muted", t.appsHint)).style.marginTop = ".6em";
}
function paintSet(bd) {
  const t = S(), tabs = h("div", "tabs");
  [[voiceOn ? t.voiceOn : t.voiceOff, "set:voice", voiceOn], [paused ? t.resume : t.pause, "set:pause", paused], [demoOn ? t.demoOn : t.demoOff, "set:demo", demoOn],
   [stereo ? t.mono : t.stereo, "set:stereo", stereo], [t.flip, "set:flip", false], [t.lang, "set:lang", false]].forEach(([l, a, on]) => tabs.appendChild(h("div", "tab" + (on ? " on" : ""), l, { act: a })));
  bd.appendChild(tabs); bd.appendChild(h("div", "muted", t.apps));
  const row = h("div", "srow"); row.append(fieldEl("newapp", t.newAppPh), h("div", "tab on", t.addBtn, { act: "set:addapp" })); bd.appendChild(row);
  customApps.forEach((a) => { const r = h("div", "srow"); r.append(h("div", "it", "🔹 " + a.n), h("div", "tab", "✕", { act: "set:del:" + a.id })); r.firstChild.style.flex = "1"; bd.appendChild(r); });
}
function paintChess(win, bd) {
  const t = S(), tabs = h("div", "tabs");
  [["w", t.white], ["b", t.black]].forEach(([k, l]) => tabs.appendChild(h("div", "tab" + (win.side === k ? " on" : ""), l, { act: "chess:side:" + k })));
  tabs.appendChild(h("div", "tab" + (win.auto ? " on" : ""), t.auto, { act: "chess:auto" }));
  bd.appendChild(tabs);
  const r = win.res;
  if (!r) bd.appendChild(h("div", "muted", t.chessHint));
  else if (r.msg) bd.appendChild(h("div", "muted", r.msg));
  else {
    bd.append(h("div", "muted", t.best), h("div", "mv", r.best), h("div", null, r.why));
    if (r.danger) bd.appendChild(h("div", "muted", t.danger + ": " + r.danger));
    if (r.conf) bd.appendChild(h("div", "muted", t.conf + ": " + r.conf));
  }
  const rep = h("div", "rep"); rep.appendChild(h("button", null, t.analyzeBoard, { act: "chess:run" })); bd.appendChild(rep);
}
function paintApp(win, bd) {
  const a = customApps.find((x) => x.id === win.app); if (!a) return;
  bd.appendChild(h("div", "muted", a.p));
  if (win.out) bd.appendChild(h("div", "it", win.out));
  const rep = h("div", "rep"); rep.appendChild(h("button", null, S().appRun, { act: "app:run" })); bd.appendChild(rep);
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
const tfetch = async (u, ms = 6000) => { const c = new AbortController(), t = setTimeout(() => c.abort(), ms); try { return await fetch(u, { signal: c.signal }); } finally { clearTimeout(t); } };
async function searchYT(q) {
  const enc = encodeURIComponent(q);
  if (ytKey) { const j = await (await tfetch(`https://www.googleapis.com/youtube/v3/search?part=snippet&type=video&maxResults=8&q=${enc}&key=${ytKey}`)).json(); if (j.error) throw new Error("yt");
    return (j.items || []).map((i) => ({ id: i.id.videoId, title: i.snippet.title })); }
  for (const b of ["https://pipedapi.kavin.rocks", "https://api.piped.private.coffee", "https://pipedapi.adminforge.de"]) {
    try { const j = await (await tfetch(`${b}/search?q=${enc}&filter=videos`)).json(); const r = (j.items || []).map((x) => ({ id: parseYT(x.url || ""), title: x.title || "" })).filter((x) => x.id); if (r.length) return r.slice(0, 8); } catch (e) {}
  }
  try { const j = await (await tfetch(`https://yewtu.be/api/v1/search?q=${enc}&type=video`)).json(); const r = (Array.isArray(j) ? j : []).map((x) => ({ id: x.videoId, title: x.title || "" })).filter((x) => x.id); if (r.length) return r.slice(0, 8); } catch (e) {}
  throw new Error("none");
}
async function ytSearch(win) {
  const q = (fields["q" + win.id] || "").trim(); if (!q || win.sbusy) return;
  win.sbusy = true; win.serr = ""; win.srv++; rp(win);
  try { win.items = await searchYT(q); win.items.forEach((i) => (titles[i.id] = i.title)); if (!win.items.length) win.serr = S().noRes; } catch (e) { win.items = []; win.serr = S().errSearch; }
  win.sbusy = false; win.srv++; rp(win);
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
  const key = (win.vid || "list") + "|" + lang + "|" + ytIds.join() + "|" + Object.values(titles).join("").length + "|" + win.srv;
  if (win.vid) { if (el._vid === win.vid) return; }
  else if (el._key === key) return;
  el._vid = win.vid; el._key = key;
  try { win.pl[e.i]?.destroy?.(); } catch (_) {} win.pl[e.i] = null;
  bd.innerHTML = "";
  if (!win.vid) {
    const t = S(), row = h("div", "srow"); row.append(fieldEl("q" + win.id, t.searchPh), h("div", "tab on", t.searchBtn, { act: "yt:search" })); bd.appendChild(row);
    if (win.sbusy) bd.appendChild(h("div", "muted", t.searching));
    else if (win.serr) bd.appendChild(h("div", "muted", win.serr));
    else if (!win.items.length) bd.appendChild(h("div", "muted", t.recent));
    const list = win.items.length ? win.items : ytIds.map((id) => ({ id }));
    const g = h("div", "grid");
    if (!win.sbusy) list.slice(0, 8).forEach(({ id }) => {
      fetchTitle(id); const v = h("div", "vid", null, { act: "yt:pick:" + id }), img = h("img"); img.src = "https://i.ytimg.com/vi/" + id + "/hqdefault.jpg"; img.alt = "";
      v.append(img, h("span", null, titles[id] && titles[id] !== "…" ? titles[id] : (id === "jNQXAC9IVRw" ? t.demoVid : "▶"))); g.appendChild(v);
    });
    bd.appendChild(g); updateFields(); return;
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
let toastList = [], toastId = 1;
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
      el.append(av, tx, h("div", "pb2")); e.toasts.appendChild(el);
    });
  });
}
function randomIncoming() { const ids = Object.keys(S().names), cid = ids[Math.floor(Math.random() * ids.length)], pool = S().msgs[cid]; incoming(cid, pool[Math.floor(Math.random() * pool.length)]); }
setInterval(() => { if (started && demoOn && !arHidden) randomIncoming(); }, 28000);
function sendReply(cid, text) { threads[cid].push({ me: true, text }); paintAll("chat"); }
function listenOnce(cb) {
  const SR = window.SpeechRecognition || window.webkitSpeechRecognition; if (!SR) return;
  try { const r = new SR(); r.lang = rtl() ? "ar-EG" : "en-US"; r.onresult = (ev) => { const t = ev.results[0][0].transcript; if (t) cb(t); }; r.start(); } catch (e) {}
}

/* system pod + dock visibility */
navigator.getBattery?.().then((b) => { const u = () => (batt = Math.round(b.level * 100)); u(); b.addEventListener("levelchange", u); }).catch(() => {});
setInterval(() => { const t = new Date().toLocaleTimeString("en-GB", { hour: "2-digit", minute: "2-digit" });
  const s = t + (batt != null ? "   🔋 " + batt + "%" : "") + (navigator.onLine ? "   📶" : ""); E.forEach((e) => { e.pod.textContent = s; e.dock.style.display = handStatus === "ready" ? "none" : "flex"; }); }, 700);

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
let handPts = [], hands21 = [], handBoxes = [], touchPts = {}, lastT = -1; const smoothMap = {};
function processHands() {
  if (!landmarker || video.readyState < 2 || video.currentTime === lastT) return; lastT = video.currentTime;
  let res; try { res = landmarker.detectForVideo(video, performance.now()); } catch (e) { return; }
  const { ew, eh } = dims(); handPts = []; hands21 = []; handBoxes = [];
  (res.landmarks || []).forEach((lm, i) => {
    const size = d2(lm[0], lm[9]) || 0.1, pinch = d2(lm[4], lm[8]) / size, raw = vToE((lm[4].x + lm[8].x) / 2, (lm[4].y + lm[8].y) / 2, ew, eh), id = "h" + i, prev = smoothMap[id];
    const x = prev ? prev.x + (raw.x - prev.x) * 0.55 : raw.x, y = prev ? prev.y + (raw.y - prev.y) * 0.55 : raw.y, down = prev && prev.down ? pinch < 0.55 : pinch < 0.38;
    smoothMap[id] = { x, y, down }; let ext = 0;
    [[8, 6], [12, 10], [16, 14], [20, 18]].forEach(([t, p]) => { if (d2(lm[t], lm[0]) > d2(lm[p], lm[0]) * 1.08) ext++; });
    handPts.push({ id, x, y, down, ext, wrist: vToE(lm[0].x, lm[0].y, ew, eh) }); hands21.push(lm.map((q) => vToE(q.x, q.y, ew, eh)));
    const xs = lm.map((q) => q.x), ys = lm.map((q) => q.y); handBoxes.push([Math.min(...xs) - 0.08, Math.min(...ys) - 0.08, Math.max(...xs) + 0.08, Math.max(...ys) + 0.08]);
  });
  Object.keys(smoothMap).forEach((k) => { if (!handPts.find((p) => p.id === k)) delete smoothMap[k]; });
}

/* =================== pin to wall: camera motion tracking =================== */
const MG = [96, 54], mcv = document.createElement("canvas"); mcv.width = MG[0]; mcv.height = MG[1];
const mctx = mcv.getContext("2d", { willReadFrequently: true });
let prevG = null, prevMask = null, lastMT = -1;
function trackMotion() {
  if (video.readyState < 2 || video.currentTime === lastMT) return; lastMT = video.currentTime;
  if (!wins.some((w) => w.pin)) { prevG = null; return; }
  const [GW, GH] = MG; mctx.drawImage(video, 0, 0, GW, GH); const d = mctx.getImageData(0, 0, GW, GH).data, g = new Uint8Array(GW * GH), mask = new Uint8Array(GW * GH);
  for (let i = 0; i < g.length; i++) g[i] = (d[i * 4] * 77 + d[i * 4 + 1] * 150 + d[i * 4 + 2] * 29) >> 8;
  handBoxes.forEach((b) => { const x0 = clamp(Math.floor(b[0] * GW), 0, GW - 1), x1 = clamp(Math.ceil(b[2] * GW), 0, GW - 1), y0 = clamp(Math.floor(b[1] * GH), 0, GH - 1), y1 = clamp(Math.ceil(b[3] * GH), 0, GH - 1);
    for (let y = y0; y <= y1; y++) for (let x = x0; x <= x1; x++) mask[y * GW + x] = 1; });
  if (prevG) {
    const R = 5, M = 8; let best = Infinity, bx = 0, by = 0, s0 = Infinity, cnt0 = 0;
    for (let sy = -R; sy <= R; sy++) for (let sx = -R; sx <= R; sx++) {
      let sad = 0, cnt = 0;
      for (let y = M; y < GH - M; y += 2) for (let x = M; x < GW - M; x += 2) {
        const ci = y * GW + x, pi = (y - sy) * GW + (x - sx); if (mask[ci] || prevMask[pi]) continue;
        sad += Math.abs(g[ci] - prevG[pi]); cnt++;
      }
      if (cnt < 120) continue; sad /= cnt;
      if (sx === 0 && sy === 0) { s0 = sad; cnt0 = cnt; }
      if (sad < best) { best = sad; bx = sx; by = sy; }
    }
    if (cnt0 && s0 - best > 1.5 && (bx || by)) {
      const { ew, eh } = dims(), vw = video.videoWidth, vh = video.videoHeight, k = Math.max(ew / vw, eh / vh);
      let dx = bx * (vw / GW) * k, dy = by * (vh / GH) * k; if (mirror) dx = -dx;
      wins.forEach((w) => {
        if (!w.pin || (zoom && zoom.id === w.id) || Object.values(press).some((s) => s.win === w)) return;
        w.x = clamp(w.x + dx / ew, -1.5, 2.2); w.y = clamp(w.y + dy / eh, -1.5, 2.2); geo(w);
      });
    }
  }
  prevG = g; prevMask = mask;
}

/* =================== interaction =================== */
const press = {}; let zoom = null, fistHold = 0, gestureLock = false, startedAt = 0, everMoved = false;
const menu = { on: false, x: 0, y: 0, s: 0, hold: 0, until: 0 };
function pick(x, y) {
  const els = document.elementsFromPoint(x, y), act = els.find((n) => n.dataset && n.dataset.act), we = els.find((n) => n.classList && n.classList.contains("win")), rz = els.some((n) => n.classList && n.classList.contains("rz"));
  return { act: act ? act.dataset.act : null, win: we ? getWin(+we.dataset.id) : null, rz };
}
function updateMenu(dt) {
  const now = performance.now(), i = handPts.findIndex((p) => !p.down && p.ext === 4);
  if (i >= 0 && !arHidden) { menu.hold += dt; if (menu.hold > 0.3) { const L = hands21[i], sz = d2(L[12], L[0]); menu.x = L[12].x; menu.y = L[12].y - sz * 0.25; menu.s = sz; menu.on = true; menu.until = now + 5000; } }
  else menu.hold = 0;
  if (menu.on && i < 0 && now > menu.until) menu.on = false;
  const r = clamp(menu.s * 1.1, 70, 180), sz = clamp(menu.s * 0.62, 40, 68);
  active().forEach((e) => {
    e.pm.style.display = menu.on ? "block" : "none"; if (!menu.on) return;
    [...e.pm.children].forEach((b, k) => {
      const th = ((200 - k * (220 / (MENU.length - 1))) * Math.PI) / 180, x = clamp(menu.x + r * Math.cos(th), sz / 2, e.ew - sz / 2), y = clamp(menu.y - r * Math.sin(th), sz / 2, e.eh - sz / 2);
      b.style.width = b.style.height = sz + "px"; b.style.fontSize = sz * 0.42 + "px"; b.style.transform = `translate(${x - sz / 2}px,${y - sz / 2}px)`;
    });
  });
}
function interact(dt) {
  processHands(); trackMotion();
  const pts = [...handPts, ...Object.values(touchPts)], seen = new Set(), { ew, eh } = dims(), hd = handPts.filter((p) => p.down);
  if (hd.length === 2 && !arHidden) {
    const a = hd[0], b = hd[1], d = d2(a, b);
    if (!zoom) { const w = pick((a.x + b.x) / 2, (a.y + b.y) / 2).win || pick(a.x, a.y).win || pick(b.x, b.y).win; if (w) { zoom = { id: w.id, d0: Math.max(d, 30), w0: w.w }; w.z = ++zTop; Object.keys(press).forEach((k) => delete press[k]); } }
    else { const w = getWin(zoom.id); if (w) { w.w = clamp(zoom.w0 * (d / zoom.d0), 0.14, 0.97); geo(w); everMoved = true; } }
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
      if (st.win && st.mode === "resize") { st.win.w = clamp((p.x - st.win.x * ew) / ew, 0.14, 0.97); geo(st.win); everMoved = true; }
    } else if (!p.down && st) {
      if (st.mode === "click" && !st.moved && performance.now() - st.t0 < 1500) doAction(st.act, st.win);
      delete press[p.id];
    }
  });
  Object.keys(press).forEach((k) => { if (!seen.has(k)) delete press[k]; });
  const h0 = handPts[0];
  if (h0 && !h0.down) {
    fistHold = h0.ext === 0 ? fistHold + dt : 0;
    if (!gestureLock && fistHold > 1.2) { gestureLock = true; arHidden = !arHidden; active().forEach((e) => e.ar.classList.toggle("ghost", arHidden)); }
  } else fistHold = 0;
  if (!h0 || h0.ext !== 0) gestureLock = false;
  updateMenu(dt);
}
function doAction(act, win) {
  const p = act.split(":"), a = p[0], b = p[1], c = p.slice(2).join(":");
  if (a === "add") {
    const fromMenu = menu.on, n = wins.length, w = addWin(b);
    if (fromMenu && wins.length > n) { const { ew, eh } = dims(); w.x = clamp(menu.x / ew - w.w / 2, 0, 1 - w.w); w.y = clamp(menu.y / eh - 0.12, 0.04, 0.7); geo(w); }
    menu.on = false; menu.until = 0; return;
  }
  if (a === "kb") return kbAct(b, c);
  if (a === "toast") { toastList = []; renderToasts(); const w = addWin("chat"); w.cid = b; paintAll("chat"); return; }
  if (a === "app" && b === "open") return addWin("app", c);
  if (a === "set") {
    if (b === "voice") voiceOn = !voiceOn; else if (b === "pause") paused = !paused; else if (b === "demo") demoOn = !demoOn;
    else if (b === "stereo") return setStereo(!stereo);
    else if (b === "flip") { facing = facing === "environment" ? "user" : "environment"; startCam(); return; }
    else if (b === "lang") { lang = lang === "ar" ? "en" : "ar"; scene = { title: "", summary: "", text: "" }; objs = []; lastAnalysis = 0; lastSig = null; }
    else if (b === "del") { customApps = customApps.filter((x) => x.id !== c); saveLS("sg_apps", customApps); paintAll("library"); }
    else if (b === "addapp") addAppFromField();
    return refreshUI();
  }
  if (!win) return;
  if (a === "close") { if (win.type === "keyboard") return closeKb(); return removeWin(win.id); }
  if (a === "pin") { win.pin = !win.pin; prevG = null; return geo(win); }
  if (a === "big") { win.w = Math.min(0.97, win.w * 1.22); return geo(win); }
  if (a === "small") { win.w = Math.max(0.14, win.w / 1.22); return geo(win); }
  if (a === "an") { if (b === "mic") return listenOnce((t) => analyze(t)); if (b === "now") return analyze(); }
  if (a === "chess") {
    if (b === "side") { win.side = c; win.res = null; return rp(win); }
    if (b === "auto") { win.auto = !win.auto; return rp(win); }
    if (b === "run") return chessRun(win);
  }
  if (a === "app" && b === "run") return runApp(win);
  if (a === "chat") {
    if (b === "open") { win.cid = c; return paintAll("chat"); }
    if (b === "reply") return sendReply(win.cid, S().reply[+c]);
    if (b === "mic") return listenOnce((t) => sendReply(win.cid, t));
  }
  if (a === "yt") {
    const p0 = win.pl[0];
    if (b === "search") return ytSearch(win);
    if (b === "pick") { win.vid = c; ytIds = [c, ...ytIds.filter((x) => x !== c)].slice(0, 12); saveLS("sg_yt", ytIds); return rp(win); }
    if (b === "back") { win.vid = null; return rp(win); }
    if (!p0 || !p0.getPlayerState) return;
    try {
      if (b === "toggle") { const st = p0.getPlayerState(); [0, 1].forEach((i) => { const q = win.pl[i]; if (q) (st === 1 ? q.pauseVideo() : q.playVideo()); }); }
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

/* =================== AI =================== */
let scene = { title: "", summary: "", text: "" }, objs = [], status = "idle", busy = false, lastAnalysis = 0, lastSig = null, lastSpoken = "";
const tiny = document.createElement("canvas"); tiny.width = 24; tiny.height = 24;
function signature() { if (video.readyState < 2) return null; const c = tiny.getContext("2d"); c.drawImage(video, 0, 0, 24, 24); const d = c.getImageData(0, 0, 24, 24).data, o = new Uint8Array(576); for (let i = 0; i < 576; i++) o[i] = (d[i * 4] + d[i * 4 + 1] + d[i * 4 + 2]) / 3; return o; }
const meanDiff = (a, b) => { let s = 0; for (let i = 0; i < a.length; i++) s += Math.abs(a[i] - b[i]); return s / a.length; };
function grabFrame(max = 1024) { const w = video.videoWidth, hh = video.videoHeight; if (!w || !hh) return null; const k = Math.min(1, max / Math.max(w, hh)), c = document.createElement("canvas"); c.width = Math.round(w * k); c.height = Math.round(hh * k); c.getContext("2d").drawImage(video, 0, 0, c.width, c.height); return c.toDataURL("image/jpeg", 0.8).split(",")[1]; }
const num = (v) => { v = Number(v); return isFinite(v) ? clamp(v, 0, 1) : null; };
async function callAI(system, text, frame) {
  const headers = { "Content-Type": "application/json" };
  if (apiKey) { headers["x-api-key"] = apiKey; headers["anthropic-version"] = "2023-06-01"; headers["anthropic-dangerous-direct-browser-access"] = "true"; }
  const res = await fetch("https://api.anthropic.com/v1/messages", { method: "POST", headers, body: JSON.stringify({ model: "claude-sonnet-4-6", max_tokens: 1000, system,
    messages: [{ role: "user", content: [{ type: "image", source: { type: "base64", media_type: "image/jpeg", data: frame } }, { type: "text", text }] }] }) });
  const j = await res.json(); if (j.error) { const er = new Error("api"); er.auth = res.status === 401; throw er; }
  return (j.content || []).map((b) => b.text || "").join("");
}
const parseJ = (raw) => JSON.parse(raw.slice(raw.indexOf("{"), raw.lastIndexOf("}") + 1));
const errKind = (e) => (!apiKey || e.auth ? "errKey" : e instanceof SyntaxError ? "errParse" : "errNet");
async function analyze(q) {
  if (typeof q !== "string") q = "";
  if (busy || !camOk) return; const frame = grabFrame(); if (!frame) return; busy = true; status = "analyzing"; const sig = signature();
  try {
    const txt = q ? (rtl() ? `سؤال المستخدم: ${q}. اجعل summary إجابة مباشرة عنه.` : `User question: ${q}. Make summary the direct answer.`) : (rtl() ? "حلل ما أراه الآن." : "Analyze what I see now.");
    const o = parseJ(await callAI(S().system, txt, frame));
    objs = (o.objects || []).slice(0, 6).map((ob) => { const b = Array.isArray(ob.box) ? ob.box.map(num) : []; return b.length < 4 || b.some((v) => v === null) ? null : { label: String(ob.label || ""), note: String(ob.note || ""), b }; }).filter(Boolean);
    scene = { title: String(o.title || ""), summary: String(o.summary || ""), text: String(o.text || "") }; status = "live"; paintAll("analysis");
    if (q) speak(scene.summary); else if (voiceOn && scene.title && scene.title !== lastSpoken) { lastSpoken = scene.title; speak(scene.title + ". " + scene.summary); }
  } catch (e) { status = errKind(e); }
  finally { busy = false; lastAnalysis = performance.now(); lastSig = sig; }
}
async function chessRun(win) {
  if (win.busy || !camOk) return; const frame = grabFrame(1400); if (!frame) return;
  win.busy = true; win.res = { msg: S().reading }; rp(win);
  try {
    const j = parseJ(await callAI(S().chessSys(win.side), S().chessAsk, frame));
    win.res = j.board ? { best: String(j.best || ""), why: String(j.why || ""), danger: String(j.danger || ""), conf: String(j.conf || "") } : { msg: String(j.why || S().noBoard) };
    if (j.board && voiceOn) speak(S().bestIs + " " + j.best + ". " + j.why);
  } catch (e) { win.res = { msg: S()[errKind(e)] }; }
  win.busy = false; win.last = performance.now(); rp(win);
}
async function runApp(win) {
  const a = customApps.find((x) => x.id === win.app); if (!a || win.busy || !camOk) return; const frame = grabFrame(); if (!frame) return;
  win.busy = true; win.out = S().analyzing; rp(win);
  try { win.out = (await callAI(S().appSys + " " + a.p, a.p, frame)).trim(); if (voiceOn) speak(win.out); } catch (e) { win.out = S()[errKind(e) === "errKey" ? "errKey" : "errNet"]; }
  win.busy = false; rp(win);
}
setInterval(() => { wins.forEach((w) => { if (w.type === "chess" && w.auto && !w.busy && performance.now() - w.last > 12000) chessRun(w); }); }, 1500);
setInterval(() => { if (!started || paused || busy || !camOk || !wins.some((w) => w.type === "analysis")) return; const since = performance.now() - lastAnalysis, sig = signature(); if (!sig) return;
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
  } else { c.fillStyle = "#0B1B4D"; c.fillRect(0, 0, ew, eh); if (started && !camOk) { c.fillStyle = "#B8C2D4"; c.font = font(15 * s); c.textAlign = "center"; c.fillText(S().noCam, ew / 2, eh / 2); c.textAlign = "start"; } }
  const g = c.createRadialGradient(ew / 2, eh / 2, Math.min(ew, eh) * 0.35, ew / 2, eh / 2, Math.max(ew, eh) * 0.8); g.addColorStop(0, "rgba(5,11,36,0)"); g.addColorStop(1, "rgba(5,11,36,.5)"); c.fillStyle = g; c.fillRect(0, 0, ew, eh);
  if (!started) return;
  objs.forEach((o) => {
    const p1 = vToE(o.b[0], o.b[1], ew, eh), p2 = vToE(o.b[0] + o.b[2], o.b[1] + o.b[3], ew, eh), x = Math.min(p1.x, p2.x), y = Math.min(p1.y, p2.y), w = Math.abs(p2.x - p1.x), hh = Math.abs(p2.y - p1.y), L = Math.min(16 * s, w / 3, hh / 3);
    c.strokeStyle = "rgba(255,255,255,.9)"; c.lineWidth = 2; c.beginPath();
    [[x, y, 1, 1], [x + w, y, -1, 1], [x, y + hh, 1, -1], [x + w, y + hh, -1, -1]].forEach(([px, py, dx, dy]) => { c.moveTo(px + dx * L, py); c.lineTo(px, py); c.lineTo(px, py + dy * L); }); c.stroke();
    c.font = font(12 * s, 600); const tw = c.measureText(o.label).width + 14 * s, lx = clamp(x, 4, ew - tw - 4), ly = Math.max(4, y - 22 * s);
    rr(c, lx, ly, tw, 20 * s, 10 * s); c.fillStyle = "rgba(0,100,224,.9)"; c.fill(); c.fillStyle = "#fff"; c.fillText(o.label, R ? lx + tw - 7 * s : lx + 7 * s, ly + 14 * s);
  });
  hands21.forEach((pts) => { c.strokeStyle = "rgba(255,255,255,.5)"; c.lineWidth = 2; c.beginPath(); LINES.forEach(([a, b]) => { c.moveTo(pts[a].x, pts[a].y); c.lineTo(pts[b].x, pts[b].y); }); c.stroke(); c.fillStyle = "#fff"; pts.forEach((p) => { c.beginPath(); c.arc(p.x, p.y, 2.4, 0, 7); c.fill(); }); });
  handPts.forEach((p) => {
    const gr = c.createLinearGradient(p.wrist.x, p.wrist.y, p.x, p.y); gr.addColorStop(0, "rgba(255,255,255,0)"); gr.addColorStop(1, "rgba(255,255,255,.9)");
    c.strokeStyle = gr; c.lineWidth = 3; c.beginPath(); c.moveTo(p.wrist.x, p.wrist.y); c.lineTo(p.x, p.y); c.stroke();
  });
  const label = paused ? S().paused : (S()[status] || "");
  const hs = handStatus === "loading" ? S().handsLoading : handStatus === "failed" ? S().handsFailed : handStatus === "ready" ? S().handsReady + (handPts.length ? " ✋" + handPts.length : "") : "";
  const sx = R ? ew - 12 : 12, top = (stereo ? 20 : 30) * s; c.font = font(12 * s, 600);
  c.fillStyle = status.startsWith("err") ? "#FF4D6A" : status === "analyzing" ? "#FFC857" : "#3DDC97"; c.beginPath(); c.arc(R ? sx - 4 : sx + 4, top - 4, 4, 0, 7); c.fill();
  c.fillStyle = "#fff"; c.fillText(label, R ? sx - 14 : sx + 14, top); c.fillStyle = "#B8C2D4"; c.font = font(11 * s); c.fillText(hs, sx, top + 16 * s);
  if (!everMoved && (performance.now() - startedAt) / 1000 < 30 && wins.length) {
    c.textAlign = "center"; c.font = font(12.5 * s, 600); const t = S().hint, tw = Math.min(ew - 16, c.measureText(t).width + 26); rr(c, (ew - tw) / 2, eh - 22 * s - 6, tw, 28 * s, 14 * s); c.fillStyle = "rgba(0,100,224,.9)"; c.fill(); c.fillStyle = "#fff"; c.fillText(t, ew / 2, eh - 3 * s - 6); c.textAlign = "start";
  }
}
function updateCursors() {
  const pts = [...handPts, ...Object.values(touchPts)], h0 = handPts[0], prog = Math.min(1, fistHold / 1.2);
  active().forEach((e) => {
    e.curs.forEach((c, k) => { const p = pts[k]; if (!p) { c.style.display = "none"; return; } c.style.display = "block"; c.style.transform = `translate(${p.x}px,${p.y}px)`; c.classList.toggle("down", p.down); });
    if (h0 && fistHold > 0.15) { e.ring.style.display = "block"; e.ring.style.transform = `translate(${h0.wrist.x}px,${h0.wrist.y}px)`; e.ring.style.background = `conic-gradient(#FF4D6A ${prog * 360}deg, rgba(255,255,255,.18) 0)`; }
    else e.ring.style.display = "none";
  });
}
let lastFrame = performance.now();
function frame(now) { const dt = Math.min(0.1, (now - lastFrame) / 1000); lastFrame = now; if (started) interact(dt); active().forEach(drawCam); updateCursors(); requestAnimationFrame(frame); }

/* =================== UI =================== */
function setStereo(v) {
  stereo = v;
  if (v) { layoutEyes(); wins.forEach((w) => ensureEl(w, E[1])); }
  else { wins.forEach((w) => { try { w.pl[1]?.destroy?.(); } catch (_) {} w.pl[1] = null; }); Object.keys(E[1].els).forEach((k) => E[1].els[k].remove()); E[1].els = {}; E[1].toasts.innerHTML = ""; layoutEyes(); }
  renderToasts(); refreshUI(); wins.forEach(geo);
}
function buildDock() {
  active().forEach((e) => {
    e.dock.innerHTML = ""; e.dock.dir = rtl() ? "rtl" : "ltr";
    [["add:library", "▦", "library"], ["add:analysis", "✦", "analysis"], ["add:youtube", "▶", "youtube"], ["add:chat", "💬", "chat"], ["add:chess", "♞", "chess"], ["add:settings", "⚙", "settings"]].forEach(([a, ic, k]) => {
      const b = h("button", "", null, { act: a }); b.append(h("b", null, ic), h("span", null, S().dock[k])); e.dock.appendChild(b);
    });
    [...e.pm.children].forEach((b, i) => (b.lastChild.textContent = S().dock[MENU[i][0]]));
  });
}
function refreshUI() {
  const t = S(); document.documentElement.lang = lang; document.documentElement.dir = rtl() ? "rtl" : "ltr";
  $("sTitle").textContent = t.title; $("sSub").textContent = t.sub; $("go").textContent = t.go; $("sLang").textContent = t.lang; $("kl1").textContent = t.kl1; $("kl2").textContent = t.kl2;
  $("sSteps").innerHTML = ""; t.steps.forEach((x) => $("sSteps").appendChild(h("div", null, x)));
  buildDock(); renderToasts(); wins.forEach((w) => { active().forEach((e) => { const el = e.els[w.id]; if (el) { delete el._key; paint(w, e); } }); });
  active().forEach((e) => (e.ar.dir = rtl() ? "rtl" : "ltr")); updateFields();
}
$("keyIn").value = apiKey; $("ytKeyIn").value = ytKey;
$("go").onclick = async () => {
  apiKey = $("keyIn").value.trim(); ytKey = $("ytKeyIn").value.trim(); saveLS("sg_key", apiKey); saveLS("sg_ytkey", ytKey);
  $("start").hidden = true; started = true; startedAt = performance.now(); beep();
  try { await document.documentElement.requestFullscreen?.(); } catch (e) {}
  try { await screen.orientation?.lock?.("landscape"); } catch (e) {}
  try { await navigator.wakeLock?.request("screen"); } catch (e) {}
  layoutEyes(); const an = addWin("analysis"); an.x = 0.03; an.y = 0.16; geo(an);
  await startCam(); loadHands(); setTimeout(() => incoming("a", S().welcome), 3500);
};
$("sLang").onclick = () => { lang = lang === "ar" ? "en" : "ar"; scene = { title: "", summary: "", text: "" }; objs = []; lastAnalysis = 0; lastSig = null; refreshUI(); };
layoutEyes(); refreshUI(); requestAnimationFrame(frame);
</script>
</body>
</html>
