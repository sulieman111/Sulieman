<!doctype html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<title>نظارة على جوالك</title>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans+Arabic:wght@400;600&display=swap" rel="stylesheet">
<style>
  :root{--ink:#0b0d12;--paper:#f3f5f9;--amber:#ffb547;--dim:rgba(243,245,249,.62)}
  *{box-sizing:border-box}
  html,body{margin:0;height:100%;background:#000;overflow:hidden;touch-action:none;
    font-family:"IBM Plex Sans Arabic",system-ui,sans-serif;color:var(--paper)}
  #view{position:fixed;inset:0;width:100%;height:100%;display:block}
  #cam{position:fixed;width:2px;height:2px;opacity:0;pointer-events:none}
  #start{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;
    padding:24px;background:var(--ink);overflow:auto}
  #start .box{max-width:460px;width:100%}
  h1{font-size:clamp(28px,6vw,40px);line-height:1.25;margin:0 0 12px;font-weight:600}
  p{margin:0 0 18px;color:var(--dim);line-height:1.8;font-size:16px}
  ul{margin:0 0 22px;padding:0;list-style:none;display:grid;gap:10px}
  li{padding:12px 14px;border-radius:14px;background:rgba(255,255,255,.07);font-size:15px;line-height:1.6}
  li b{color:var(--amber);font-weight:600}
  label{display:flex;gap:10px;align-items:center;margin:0 0 18px;font-size:15px}
  input[type=checkbox]{width:20px;height:20px;accent-color:var(--amber)}
  button{width:100%;padding:16px;border:0;border-radius:16px;background:var(--amber);color:#1a1205;
    font:600 18px "IBM Plex Sans Arabic",system-ui,sans-serif;cursor:pointer}
  button:focus-visible,input:focus-visible{outline:3px solid #fff;outline-offset:3px}
  #err{color:#ff9d8a;margin-top:12px;font-size:14px;min-height:20px}
</style>
</head>
<body>
<video id="cam" playsinline muted autoplay></video>
<canvas id="view"></canvas>

<div id="start">
  <div class="box">
    <h1>جوالك صار نظارة ذكية</h1>
    <p>الكاميرا الخلفية هي عيون النظارة، وتتحكم بالواجهة بيدك فقط. ضع الجوال بوضع أفقي داخل حامل نظارة (مثل Cardboard).</p>
    <ul>
      <li><b>قرصة</b> السبابة مع الإبهام = اختيار</li>
      <li><b>قرصة</b> الوسطى مع الإبهام = رجوع</li>
      <li>حرّك يدك أمام الكاميرا لتحريك المؤشر (واللمس يعمل كبديل)</li>
    </ul>
    <label><input type="checkbox" id="stereoBox" checked> وضع النظارة (شاشتان، عين لكل جهة)</label>
    <button id="go">ابدأ</button>
    <div id="err" role="alert"></div>
    <div style="margin-top:14px;font-size:12px;color:var(--dim)">الإصدار 2</div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>
<script>
/* ============ إعدادات ============ */
const API_URL = '/api/vision';   // خادم الذكاء الاصطناعي (server.js)
const LANG = 'ar-SA';            // لغة الصوت

/* ============ أدوات ============ */
const $ = id => document.getElementById(id);
const clamp = (v,a,b) => Math.min(b, Math.max(a, v));
const cam = $('cam'), view = $('view'), vctx = view.getContext('2d');
const eye = document.createElement('canvas'), c = eye.getContext('2d');
let DPR = 1, stereo = true, running = false, P = null;
const st = {screen:'home', items:[], toast:null, flash:0, photos:[], photoIdx:0,
  aiStatus:'idle', aiText:'', aiScroll:0, capFinal:'', capTmp:'', heading:null};
const cur = {x:.5, y:.5, on:false, until:0};
let pinchIdx = false, pinchMid = false;

function rr(ctx,x,y,w,h,r){ctx.beginPath();ctx.moveTo(x+r,y);ctx.arcTo(x+w,y,x+w,y+h,r);
  ctx.arcTo(x+w,y+h,x,y+h,r);ctx.arcTo(x,y+h,x,y,r);ctx.arcTo(x,y,x+w,y,r);ctx.closePath();}
function cover(ctx,v,x,y,w,h){const vw=v.videoWidth,vh=v.videoHeight;if(!vw)return;
  const s=Math.max(w/vw,h/vh),dw=vw*s,dh=vh*s;ctx.drawImage(v,x+(w-dw)/2,y+(h-dh)/2,dw,dh);}
function txt(s,x,y,size,o={}){c.font=`${o.w||600} ${size}px "IBM Plex Sans Arabic",system-ui,sans-serif`;
  c.fillStyle=o.col||'#f3f5f9';c.textAlign=o.align||'center';c.textBaseline=o.base||'middle';c.fillText(s,x,y);}
function wrap(s,maxW){const words=s.split(/\s+/),lines=[];let line='';
  for(const w of words){const t=line?line+' '+w:w;
    if(c.measureText(t).width>maxW&&line){lines.push(line);line=w;}else line=t;}
  if(line)lines.push(line);return lines;}
function toast(t){st.toast={t,until:performance.now()+2500};}

/* ============ الشاشة والعرض ============ */
function resize(){
  DPR=Math.min(window.devicePixelRatio||1,2);
  view.width=Math.floor(innerWidth*DPR);view.height=Math.floor(innerHeight*DPR);
  eye.width=stereo?Math.floor(view.width/2):view.width;eye.height=view.height;
}
addEventListener('resize',resize);

function glass(x,y,w,h,r){
  c.save();rr(c,x,y,w,h,r);c.clip();
  if(cam.readyState>=2){try{c.filter=`blur(${Math.round(w/45)}px)`;}catch(e){}
    cover(c,cam,0,0,eye.width,eye.height);c.filter='none';}
  c.fillStyle='rgba(10,12,18,.62)';c.fillRect(x,y,w,h);c.restore();
  c.save();rr(c,x,y,w,h,r);c.strokeStyle='rgba(255,255,255,.16)';c.lineWidth=Math.max(1,DPR);c.stroke();c.restore();
}
function hit(x,y,w,h){const cx=cur.x*eye.width,cy=cur.y*eye.height;
  return (cur.on||performance.now()<cur.until)&&cx>=x&&cx<=x+w&&cy>=y&&cy<=y+h;}
function button(id,x,y,w,h,label,size,o={}){
  const hov=hit(x,y,w,h);st.items.push({id,x,y,w,h});
  c.save();rr(c,x,y,w,h,Math.min(w,h)*.28);
  c.fillStyle=o.accent?(hov?'#ffc977':'#ffb547'):(hov?'rgba(255,255,255,.26)':'rgba(255,255,255,.09)');c.fill();
  if(hov){c.strokeStyle='rgba(255,255,255,.7)';c.lineWidth=2*DPR;c.stroke();}c.restore();
  if(label)txt(label,x+w/2,y+h/2,size,{col:o.accent?'#1a1205':'#f3f5f9'});
  return hov;
}
function backBtn(){const u=P.u;button('back',P.px+u*.7,P.py+u*.7,u*3.6,u*1.6,'رجوع ›',u*.8);}

/* ============ الشاشات ============ */
const SCREENS = {
  home(){
    const {px,py,pw,ph,u}=P,now=new Date();
    txt(now.toLocaleTimeString('en-GB',{hour:'2-digit',minute:'2-digit'}),px+pw/2,py+u*1.9,u*2.2,{w:600});
    txt(now.toLocaleDateString('ar',{weekday:'long',day:'numeric',month:'long'}),px+pw/2,py+u*3.7,u*.8,{w:400,col:'rgba(243,245,249,.65)'});
    const apps=[['camera','📷','الكاميرا'],['ai','✨','اسأل الذكاء'],['captions','🎙️','نص مباشر'],
      ['compass','🧭','البوصلة'],['gallery','🖼️','الصور'],['mode','👓',stereo?'وضع نظارة':'وضع عادي']];
    const cols=3,gap=u*.5,gx=px+u*.9,gy=py+u*4.6,gw=pw-u*1.8,gh=ph-u*5.4;
    const tw=(gw-gap*(cols-1))/cols,th=(gh-gap)/2;
    apps.forEach((a,i)=>{
      const x=gx+(i%cols)*(tw+gap),y=gy+Math.floor(i/cols)*(th+gap);
      button((a[0]==='mode'?'mode':'go:'+a[0]),x,y,tw,th,'',0);
      txt(a[1],x+tw/2,y+th*.38,Math.min(u*1.7,th*.45));
      txt(a[2],x+tw/2,y+th*.78,Math.min(u*.72,th*.2),{w:400});
    });
  },
  camera(){
    const w=eye.width,h=eye.height,u=Math.min(w,h)/14,m=u*1.2,L=u*1.4;
    c.save();c.strokeStyle='rgba(255,255,255,.85)';c.lineWidth=2.5*DPR;
    [[m,m,1,1],[w-m,m,-1,1],[m,h-m,1,-1],[w-m,h-m,-1,-1]].forEach(([x,y,dx,dy])=>{
      c.beginPath();c.moveTo(x,y+dy*L);c.lineTo(x,y);c.lineTo(x+dx*L,y);c.stroke();});c.restore();
    button('back',w/2-u*7.5,m+u*.2,u*3.6,u*1.6,'رجوع ›',u*.8);
    const sx=w/2,sy=h-m-u*1.6,r=u*1.5,hov=hit(sx-r,sy-r,r*2,r*2);
    st.items.push({id:'shutter',x:sx-r,y:sy-r,w:r*2,h:r*2});
    c.beginPath();c.arc(sx,sy,r,0,7);c.lineWidth=3*DPR;c.strokeStyle='#fff';c.stroke();
    c.beginPath();c.arc(sx,sy,r*(hov?.78:.7),0,7);c.fillStyle=hov?'#ffb547':'#fff';c.fill();
    txt('اقرص للتصوير',sx,sy-r-u*.9,u*.7,{w:400,col:'rgba(255,255,255,.85)'});
  },
  ai(){
    const {px,py,pw,ph,u}=P;backBtn();
    txt('اسأل الذكاء',px+pw/2,py+u*1.5,u*1.0);
    const ax=px+u*1,aw=pw-u*2,ay=py+u*3,ah=ph-u*6.2;
    let body=st.aiText;
    if(st.aiStatus==='listening')body='أسمعك… اسأل عمّا تراه';
    else if(st.aiStatus==='thinking')body='أفكر…';
    else if(!body)body='اضغط «اسأل» ثم تكلم، وسأنظر عبر الكاميرا وأجاوبك.';
    c.font=`400 ${u*.82}px "IBM Plex Sans Arabic",system-ui,sans-serif`;
    const lh=u*1.25,lines=wrap(body,aw),cap=Math.max(1,Math.floor(ah/lh));
    st.aiScroll=clamp(st.aiScroll,0,Math.max(0,lines.length-cap));
    c.save();c.beginPath();c.rect(ax,ay,aw,ah);c.clip();
    lines.slice(st.aiScroll,st.aiScroll+cap).forEach((l,i)=>txt(l,ax+aw,ay+lh*(i+.5),u*.82,{w:400,align:'right'}));c.restore();
    if(lines.length>cap){button('up',px+u*.7,py+ph-u*2.6,u*1.6,u*1.6,'▴',u);button('down',px+u*2.5,py+ph-u*2.6,u*1.6,u*1.6,'▾',u);}
    button('ask',px+pw-u*7.2,py+ph-u*2.6,u*6.5,u*1.8,st.aiStatus==='idle'?'🎙️ اسأل':'…',u*.85,{accent:st.aiStatus==='idle'});
  },
  captions(){
    const {u}=P,w=eye.width,h=eye.height,bw=Math.min(w*.9,u*26),bh=Math.min(h*.4,u*8),bx=(w-bw)/2,by=h-bh-h*.06;
    glass(bx,by,bw,bh,u*1.1);
    button('back',bx+u*.6,by+u*.6,u*3.4,u*1.5,'رجوع ›',u*.75);
    const full=(st.capFinal+' '+st.capTmp).trim()||'تكلم… سأكتب كلامك هنا مباشرة';
    c.font=`400 ${u*.95}px "IBM Plex Sans Arabic",system-ui,sans-serif`;
    const lines=wrap(full,bw-u*1.6).slice(-3),lh=u*1.35;
    lines.forEach((l,i)=>txt(l,bx+bw-u*.8,by+bh-u*.8-(lines.length-1-i)*lh-lh/2,u*.95,{w:400,align:'right'}));
  },
  compass(){
    const {px,py,pw,ph,u}=P;backBtn();
    const hd=st.heading,cx=px+pw/2,cy=py+ph*.52,R=Math.min(pw,ph)*.3;
    c.save();c.strokeStyle='rgba(255,255,255,.35)';c.lineWidth=2*DPR;c.beginPath();c.arc(cx,cy,R,0,7);c.stroke();
    if(hd!==null){c.translate(cx,cy);c.rotate(-hd*Math.PI/180);
      c.fillStyle='#ffb547';c.beginPath();c.moveTo(0,-R*.85);c.lineTo(R*.14,0);c.lineTo(-R*.14,0);c.closePath();c.fill();
      c.fillStyle='#fff';c.beginPath();c.moveTo(0,R*.85);c.lineTo(R*.14,0);c.lineTo(-R*.14,0);c.closePath();c.fill();}
    c.restore();
    const names=['شمال','شمال شرق','شرق','جنوب شرق','جنوب','جنوب غرب','غرب','شمال غرب'];
    txt(hd===null?'حرّك الجوال…':Math.round(hd)+'°  '+names[Math.round(hd/45)%8],cx,py+ph-u*1.6,u*1.2);
  },
  gallery(){
    const {px,py,pw,ph,u}=P;backBtn();
    txt('الصور ('+st.photos.length+')',px+pw/2,py+u*1.5,u*1.0);
    if(!st.photos.length){txt('لا توجد صور بعد',px+pw/2,py+ph/2,u*.95,{w:400,col:'rgba(243,245,249,.65)'});return;}
    const im=st.photos[st.photoIdx],ix=px+u*1.2,iy=py+u*3,iw=pw-u*2.4,ih=ph-u*6;
    const s=Math.min(iw/im.width,ih/im.height),dw=im.width*s,dh=im.height*s;
    c.drawImage(im,ix+(iw-dw)/2,iy+(ih-dh)/2,dw,dh);
    button('prev',px+u*.9,py+ph-u*2.5,u*3.4,u*1.7,'‹ السابق',u*.8);
    button('next',px+pw-u*4.3,py+ph-u*2.5,u*3.4,u*1.7,'التالي ›',u*.8);
  }
};

function draw(){
  const w=eye.width,h=eye.height;st.items=[];
  c.clearRect(0,0,w,h);
  if(cam.readyState>=2)cover(c,cam,0,0,w,h);else{c.fillStyle='#10131a';c.fillRect(0,0,w,h);}
  const pw=Math.min(w*.84,h*1.3),ph=Math.min(h*.72,pw*.8),px=(w-pw)/2,py=h>w?h-ph-h*.1:(h-ph)/2;
  P={px,py,pw,ph,u:pw/20};
  if(['home','ai','compass','gallery'].includes(st.screen))glass(px,py,pw,ph,P.u*1.2);
  SCREENS[st.screen]();
  if(st.flash>0){c.fillStyle=`rgba(255,255,255,${st.flash})`;c.fillRect(0,0,w,h);st.flash-=.06;}
  if(st.toast&&performance.now()<st.toast.until){const u=P.u;c.font=`600 ${u*.8}px "IBM Plex Sans Arabic",system-ui,sans-serif`;
    const tw=c.measureText(st.toast.t).width+u*2;c.fillStyle='rgba(10,12,18,.8)';rr(c,(w-tw)/2,u*.8,tw,u*1.7,u*.85);c.fill();
    txt(st.toast.t,w/2,u*1.65,u*.8);}
  if(cur.on||performance.now()<cur.until){const cx=cur.x*w,cy=cur.y*h,r=P.u*.55;
    c.beginPath();c.arc(cx,cy,r,0,7);
    if(pinchIdx){c.fillStyle='rgba(255,181,71,.9)';c.fill();}
    c.lineWidth=2.5*DPR;c.strokeStyle='#fff';c.stroke();}
  vctx.fillStyle='#000';vctx.fillRect(0,0,view.width,view.height);
  vctx.drawImage(eye,0,0);
  if(stereo){vctx.drawImage(eye,eye.width,0);vctx.fillStyle='#000';vctx.fillRect(eye.width-1,0,2,view.height);}
}

/* ============ التفاعل ============ */
function go(s){
  if(st.screen==='captions')stopCaptions();
  st.screen=s;
  if(s==='captions'){st.capFinal='';st.capTmp='';startCaptions();}
  if(s==='ai'){st.aiScroll=0;askAI();}
}
function act(id){
  if(id.startsWith('go:'))go(id.slice(3));
  else if(id==='back')go('home');
  else if(id==='mode'){stereo=!stereo;resize();}
  else if(id==='shutter')capture();
  else if(id==='ask'){if(st.aiStatus==='idle')askAI();}
  else if(id==='prev')st.photoIdx=(st.photoIdx-1+st.photos.length)%Math.max(1,st.photos.length);
  else if(id==='next')st.photoIdx=(st.photoIdx+1)%Math.max(1,st.photos.length);
  else if(id==='up')st.aiScroll--;
  else if(id==='down')st.aiScroll++;
}
function click(){const cx=cur.x*eye.width,cy=cur.y*eye.height;
  const it=st.items.find(i=>cx>=i.x&&cx<=i.x+i.w&&cy>=i.y&&cy<=i.y+i.h);if(it)act(it.id);}
function back(){if(st.screen!=='home')go('home');}

view.addEventListener('pointerdown',e=>{
  const x=e.clientX*DPR,y=e.clientY*DPR;
  cur.x=((stereo?x%eye.width:x))/eye.width;cur.y=y/eye.height;cur.until=performance.now()+2000;click();
});

function onHands(r){
  const L=r.multiHandLandmarks&&r.multiHandLandmarks[0];
  if(!L){cur.on=false;pinchIdx=false;pinchMid=false;return;}
  const d=(a,b)=>Math.hypot(L[a].x-L[b].x,L[a].y-L[b].y),size=d(0,9)||.1;
  const ti=d(4,8)/size,tm=d(4,12)/size;
  const tx=clamp(((L[4].x+L[8].x)/2-.12)/.76,0,1),ty=clamp(((L[4].y+L[8].y)/2-.12)/.76,0,1);
  cur.x+=(tx-cur.x)*.4;cur.y+=(ty-cur.y)*.4;cur.on=true;
  const idx=pinchIdx?ti<.42:ti<.30,mid=pinchMid?tm<.42:tm<.30;
  if(idx&&!pinchIdx)click();else if(mid&&!pinchMid&&!idx)back();
  pinchIdx=idx;pinchMid=mid;
}

/* ============ الكاميرا والصور ============ */
function snap(max){
  const vw=cam.videoWidth,vh=cam.videoHeight,s=Math.min(1,max/Math.max(vw,vh));
  const k=document.createElement('canvas');k.width=Math.round(vw*s);k.height=Math.round(vh*s);
  k.getContext('2d').drawImage(cam,0,0,k.width,k.height);return k;
}
function capture(){st.photos.push(snap(1280));st.photoIdx=st.photos.length-1;st.flash=1;toast('تم التصوير');}

/* ============ الصوت والذكاء الاصطناعي ============ */
const SR=window.SpeechRecognition||window.webkitSpeechRecognition;
let capRec=null;
function startCaptions(){
  if(!SR){toast('المتصفح لا يدعم التعرف على الصوت');return;}
  capRec=new SR();capRec.lang=LANG;capRec.continuous=true;capRec.interimResults=true;
  capRec.onresult=e=>{let f='',t='';for(let i=e.resultIndex;i<e.results.length;i++){
    const s=e.results[i][0].transcript;if(e.results[i].isFinal)f+=s;else t+=s;}
    if(f)st.capFinal=(st.capFinal+' '+f).trim().split(/\s+/).slice(-40).join(' ');st.capTmp=t;};
  capRec.onend=()=>{if(st.screen==='captions')try{capRec.start();}catch(e){}};
  try{capRec.start();}catch(e){}
}
function stopCaptions(){if(capRec){capRec.onend=null;try{capRec.stop();}catch(e){}capRec=null;}}

function listenOnce(){return new Promise(res=>{
  if(!SR)return res('');
  const r=new SR();r.lang=LANG;r.interimResults=false;let done=false;
  const fin=t=>{if(!done){done=true;res(t||'');}};
  r.onresult=e=>fin(e.results[0][0].transcript);r.onerror=()=>fin('');r.onend=()=>fin('');
  setTimeout(()=>{try{r.stop();}catch(e){}fin('');},8000);
  try{r.start();}catch(e){fin('');}
});}
function speak(t){try{speechSynthesis.cancel();const u=new SpeechSynthesisUtterance(t);u.lang=LANG;speechSynthesis.speak(u);}catch(e){}}

async function askAI(){
  if(st.aiStatus!=='idle')return;
  st.aiText='';st.aiScroll=0;st.aiStatus='listening';
  const q=await listenOnce();
  st.aiStatus='thinking';
  try{
    const img=snap(1024).toDataURL('image/jpeg',.82);
    const r=await fetch(API_URL,{method:'POST',headers:{'Content-Type':'application/json'},
      body:JSON.stringify({image:img,question:q||'صف لي باختصار ما تراه وكيف أساعدك فيه.'})});
    if(!r.ok)throw new Error(r.status);
    st.aiText=(await r.json()).text||'لم أستطع الرد.';speak(st.aiText);
  }catch(e){st.aiText='تعذر الاتصال بالخادم. شغّل server.js وضع مفتاح الـ API (راجع التعليمات).';}
  st.aiStatus='idle';
}

/* ============ البوصلة ============ */
function onOrient(e){
  if(typeof e.webkitCompassHeading==='number')st.heading=e.webkitCompassHeading;
  else if(e.absolute&&e.alpha!==null)st.heading=(360-e.alpha)%360;
}

/* ============ التشغيل ============ */
let hands=null,busy=false;
function loop(){
  if(hands&&!busy&&cam.readyState>=2){busy=true;hands.send({image:cam}).catch(()=>{}).finally(()=>busy=false);}
  draw();requestAnimationFrame(loop);
}
$('go').addEventListener('click',async()=>{
  $('err').textContent='';stereo=$('stereoBox').checked;
  try{
    if(!navigator.mediaDevices||!navigator.mediaDevices.getUserMedia)throw {name:'NoMediaDevices'};
    let stream;
    try{stream=await navigator.mediaDevices.getUserMedia({audio:false,
      video:{facingMode:{ideal:'environment'},width:{ideal:1280},height:{ideal:720}}});}
    catch(e1){
      if(e1.name==='NotAllowedError'||e1.name==='SecurityError')throw e1;
      stream=await navigator.mediaDevices.getUserMedia({audio:false,video:true});
    }
    cam.srcObject=stream;
    await cam.play();
  }catch(e){
    const n=e&&e.name,m={
      NotAllowedError:'الإذن مرفوض. اضغط على أيقونة الإعدادات يسار الرابط ← الأذونات ← الكاميرا ← سماح، ثم حدّث الصفحة.',
      NotReadableError:'الكاميرا مستخدمة من تطبيق آخر. أغلق التطبيقات التي تستخدمها وحاول مرة ثانية.',
      NotFoundError:'لم يتم العثور على كاميرا في الجهاز.',
      SecurityError:'المتصفح منع الكاميرا في هذه الصفحة. افتحها مباشرة في Chrome وليس داخل تطبيق آخر.',
      NoMediaDevices:'الصفحة لا تعمل عبر HTTPS أو مفتوحة داخل تطبيق لا يدعم الكاميرا. افتحها في Chrome.'
    };
    $('err').textContent=(m[n]||'تعذر فتح الكاميرا.')+' ('+(n||'unknown')+')';return;}
  try{if(typeof DeviceOrientationEvent!=='undefined'&&DeviceOrientationEvent.requestPermission)await DeviceOrientationEvent.requestPermission();}catch(e){}
  addEventListener('deviceorientationabsolute',onOrient,true);addEventListener('deviceorientation',onOrient,true);
  try{await document.fonts.load('600 16px "IBM Plex Sans Arabic"');}catch(e){}
  try{await document.documentElement.requestFullscreen();await screen.orientation.lock('landscape');}catch(e){}
  if(window.Hands){
    hands=new Hands({locateFile:f=>`https://cdn.jsdelivr.net/npm/@mediapipe/hands/${f}`});
    hands.setOptions({maxNumHands:1,modelComplexity:0,minDetectionConfidence:.6,minTrackingConfidence:.5});
    hands.onResults(onHands);
  }else toast('تتبع اليد غير متاح، استخدم اللمس');
  $('start').remove();resize();running=true;requestAnimationFrame(loop);
});
resize();
</script>
</body>
</html>
