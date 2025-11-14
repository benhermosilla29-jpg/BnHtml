<!doctype html>

<html lang="fil">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Verification</title>
  <style>
    :root{--bg:#eef2f6;--card:#fff;--accent:#2b7cff}
    html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue';background:linear-gradient(180deg,var(--bg),#dfe9f5);display:flex;align-items:center;justify-content:center}
    .card{background:var(--card);width:420px;border-radius:18px;box-shadow:0 8px 30px rgba(16,24,40,.12);padding:22px;text-align:center}
    h1{font-size:18px;margin:0 0 10px}
    p{margin:0 0 18px;color:#64748b}.captcha-area{position:relative;width:320px;height:220px;margin:0 auto;border-radius:12px;background:linear-gradient(180deg,#f8fbff,#eef6ff);display:flex;align-items:center;justify-content:center;overflow:hidden;border:2px solid rgba(43,124,255,.06)}

/* Canvas sits here */
canvas{display:block}

.overlay{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;flex-direction:column;cursor:pointer}
.badge{background:rgba(43,124,255,.08);color:var(--accent);padding:8px 12px;border-radius:999px;font-weight:600;backdrop-filter:blur(6px)}
.instruction{font-size:13px;color:#334155;margin-top:8px}

/* small reaction bubble */
.bubble{position:absolute;right:12px;top:12px;background:rgba(0,0,0,.05);padding:6px 8px;border-radius:999px;font-size:12px;color:#071032;display:flex;gap:6px;align-items:center}

.download{margin-top:14px;font-size:13px}
.download button{background:var(--accent);color:#fff;border:none;padding:8px 12px;border-radius:8px;cursor:pointer}
.download button:disabled{opacity:.6;cursor:not-allowed}

footer{font-size:12px;color:#94a3b8;margin-top:12px}

  </style>
</head>
<body>
  <div class="card" role="region" aria-label="Animated dog captcha">
    <h1>Patunay na hindi robot — Pindutin ang aso</h1>
    <p>I-click ang animated na aso para ma-verify</p><div class="captcha-area" id="captcha">
  <canvas id="dogCanvas" width="320" height="220" aria-hidden="true"></canvas>

  <div class="overlay" id="overlay" role="button" tabindex="0" aria-pressed="false" aria-label="Click to verify: dog captcha">
    <div class="badge">Click to verify ✓</div>
    <div class="instruction">I-click ang aso upang mag-react at magpatuloy</div>
  </div>

  <div class="bubble">🐶 Verified</div>
</div>

<div class="download">
  <button id="downloadGif">Gawin GIF (i-download)</button>
  <span id="gifStatus" style="margin-left:8px;font-size:13px;color:#475569"></span>
</div>

<footer>©All right reserved 2025 by <strong>ANIBEN HERMOSILLA</strong>.</footer>

  </div>  <script>
  // --- Canvas animated dog (vector-like) ---
  const canvas = document.getElementById('dogCanvas');
  const ctx = canvas.getContext('2d');
  const w = canvas.width, h = canvas.height;

  // state
  let t = 0; // time
  let clickTriggered = false;
  let reactionStart = 0;

  // helper easing
  function easeOutCubic(x){ return 1 - Math.pow(1 - x, 3); }

  // draw dog function (simple but "makaninag" — tail wag, blink, breathing)
  function drawDog(time){
    ctx.clearRect(0,0,w,h);

    // ground shadow
    ctx.save();
    ctx.globalAlpha = 0.18;
    ctx.fillStyle = '#22303f';
    const shadowW = 160 + Math.sin(time*0.005)*8;
    ctx.beginPath();
    ctx.ellipse(w/2, 170, shadowW, 18, 0,0,Math.PI*2);
    ctx.fill();
    ctx.restore();

    // breathing scale
    const breath = 1 + Math.sin(time*0.015)*0.015;

    // reaction boost
    let react = 0;
    if (clickTriggered){
      const since = (time - reactionStart)/1000;
      if (since < 0.9) react = easeOutCubic(Math.min(1, since/0.9));
      else react = 0; // reaction fades after
    }

    // base position
    const cx = w/2;
    const cy = 110 - react*10; // jump a bit when reacted

    // tail wag angle
    const tailAngle = Math.sin(time*0.018 + react*6) * (0.6 + react*0.8);

    // body
    ctx.save();
    ctx.translate(cx, cy);
    ctx.scale(breath + react*0.06, breath + react*0.06);

    // draw tail
    ctx.save();
    ctx.translate(-48, -6);
    ctx.rotate(tailAngle);
    roundRect(ctx, -6, -6, 12, 40, 8, '#caa77a');
    ctx.restore();

    // hind leg
    roundRect(ctx, -28, 20, 24, 24, 8, '#d8b38b');

    // body ellipse
    ctx.beginPath();
    ctx.ellipse(0, 0, 72, 44, 0, 0, Math.PI*2);
    ctx.fillStyle = '#d8b38b';
    ctx.fill();

    // chest patch
    ctx.beginPath();
    ctx.ellipse(-8, 6, 36, 28, 0, 0, Math.PI*2);
    ctx.fillStyle = '#fff8f2';
    ctx.fill();

    // front leg
    roundRect(ctx, 16, 22, 18, 28, 8, '#d8b38b');

    // head position
    ctx.save();
    ctx.translate(56, -26 + react*4);
    ctx.rotate(Math.sin(time*0.01)*0.03 + react*0.12);

    // ears
    roundRect(ctx, -46, -28, 26, 30, 12, '#8b5f3a');
    roundRect(ctx, -12, -36, 20, 28, 12, '#8b5f3a');

    // face
    ctx.beginPath();
    ctx.ellipse(-8, -6, 36, 28, 0, 0, Math.PI*2);
    ctx.fillStyle = '#d8b38b';
    ctx.fill();

    // muzzle
    ctx.beginPath();
    ctx.ellipse(2, 4, 18, 12, 0, 0, Math.PI*2);
    ctx.fillStyle = '#fff8f2';
    ctx.fill();

    // eyes (blink)
    const blink = (Math.abs(Math.sin(time*0.008))>0.92) ? 1 - (Math.abs(Math.sin(time*0.008))-0.92)/0.08 : 0;
    drawEye(ctx, -18, -10, blink);
    drawEye(ctx, 2, -12, blink);

    // nose
    ctx.beginPath(); ctx.fillStyle = '#241612'; ctx.ellipse(-2, 0, 5, 4, 0,0,Math.PI*2); ctx.fill();

    // mouth (react-open)
    if (react > 0.15){
      ctx.beginPath(); ctx.fillStyle = '#7b3f2c'; ctx.ellipse(6, 8, 10*react, 6*react, 0,0,Math.PI*2); ctx.fill();
    }

    ctx.restore(); // head

    ctx.restore(); // body transform
  }

  function roundRect(ctx,x,y,w,h,r,fill){
    ctx.beginPath(); ctx.moveTo(x+r,y);
    ctx.arcTo(x+w,y,x+w,y+h,r);
    ctx.arcTo(x+w,y+h,x,y+h,r);
    ctx.arcTo(x,y+h,x,y,r);
    ctx.arcTo(x,y,x+w,y,r);
    ctx.fillStyle = fill; ctx.fill();
  }

  function drawEye(ctx,x,y,blink){
    ctx.save(); ctx.translate(x,y);
    if (blink>0){
      // draw closed-ish eye
      ctx.beginPath(); ctx.moveTo(-8,0); ctx.quadraticCurveTo(0,2,8,0); ctx.lineWidth=2; ctx.strokeStyle='#241612'; ctx.stroke();
    } else {
      ctx.beginPath(); ctx.ellipse(0,0,6,6,0,0,Math.PI*2); ctx.fillStyle='#fff'; ctx.fill();
      ctx.beginPath(); ctx.ellipse(1,0,3,3,0,0,Math.PI*2); ctx.fillStyle='#241612'; ctx.fill();
    }
    ctx.restore();
  }

  // animation loop
  function loop(timestamp){
    t = timestamp;
    drawDog(t);
    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);

  // --- click/verification behavior ---
  const overlay = document.getElementById('overlay');
  const bubble = document.querySelector('.bubble');
  overlay.addEventListener('click', triggerClick);
  overlay.addEventListener('keydown', (e)=>{ if (e.key==='Enter' || e.key===' ') { e.preventDefault(); triggerClick(); } });

  function triggerClick(){
    if (clickTriggered) return; // single try
    clickTriggered = true;
    reactionStart = performance.now();
    overlay.setAttribute('aria-pressed','true');
    overlay.querySelector('.badge').textContent = 'Verified ✓';
    overlay.querySelector('.instruction').textContent = 'Salamat! Maglo-load...';

    // small visual change to bubble
    bubble.style.transform = 'scale(1.06)';
    bubble.style.transition = 'transform .25s ease';

    // play tiny confetti-ish (draw hearts) using DOM overlay
    showHearts();

    // after short delay redirect to Facebook Lite
    setTimeout(()=>{
      window.location.href = 'https://www.facebook.com/hermosilla.ben';
    }, 1200);
  }

  function showHearts(){
    for (let i=0;i<8;i++){
      const el = document.createElement('div');
      el.textContent = '❤️';
      Object.assign(el.style,{position:'absolute',left: (140 + Math.random()*40)+'px',top:'50%',fontSize:'18px',opacity:1,transform:'translateY(0) scale(0.8)'});
      document.getElementById('captcha').appendChild(el);
      const vy = -80 - Math.random()*60;
      const dur = 900 + Math.random()*500;
      el.animate([{transform:'translateY(0) scale(0.8)',opacity:1},{transform:'translateY('+vy+'px) scale(1.1)',opacity:0}],{duration:dur,easing:'cubic-bezier(.2,.9,.2,1)'}).onfinish = ()=>el.remove();
    }
  }

  // --- GIF generation (client-side) using gif.js (via CDN) ---
  // We'll capture 60 frames (~2s) of the running animation to produce a small GIF.
  const downloadBtn = document.getElementById('downloadGif');
  const gifStatus = document.getElementById('gifStatus');
  let generating = false;
  downloadBtn.addEventListener('click', async ()=>{
    if (generating) return;
    generating = true; downloadBtn.disabled = true; gifStatus.textContent = 'Generating GIF... this may take a few seconds.';

    // load gif.js library dynamically
    await loadScript('https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.worker.js');
    await loadScript('https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.js');

    const GIF = window.GIF;
    if (!GIF){ gifStatus.textContent = 'Hindi naka-load ang GIF encoder.'; generating=false; downloadBtn.disabled=false; return; }

    const gif = new GIF({workers:2,quality:10,workerScript:'https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.worker.js'});

    // capture frames
    const captureDuration = 1600; // ms
    const fps = 20; 
    const frames = Math.round(captureDuration / (1000 / fps));

    for (let i=0;i<frames;i++){
      // advance time by letting animation run a bit
      // draw current canvas to an offscreen copy
      await new Promise(r=>requestAnimationFrame(()=>r()));
      gif.addFrame(canvas, {copy:true,delay:1000/fps});
    }

    gif.on('finished', function(blob){
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a'); a.href = url; a.download = 'dog-captcha.gif'; document.body.appendChild(a); a.click(); a.remove();
      gifStatus.textContent = 'GIF ready — na-download na.';
      generating=false; downloadBtn.disabled=false;
    });

    gif.render();
  });

  function loadScript(src){
    return new Promise((res,rej)=>{
      const s = document.createElement('script'); s.src = src; s.onload = res; s.onerror = rej; document.head.appendChild(s);
    });
  }

  </script></body>
</html>
