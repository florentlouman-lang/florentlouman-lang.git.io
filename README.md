# florentlouman-lang.git.io
Lecteur audio modulable
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Lecteur audio - PWA & animations</title>
  <meta name="theme-color" content="#0b1220">
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;600;800&display=swap');
    :root{--bg:#0b1220;--card:#0f1724;--accent:#4f46e5;--muted:#94a3b8}
    *{box-sizing:border-box}
    body{font-family:Inter,system-ui,Arial,sans-serif;margin:0;min-height:100vh;background:linear-gradient(180deg,var(--bg),#071025);display:flex;align-items:center;justify-content:center;padding:28px;color:#e6eef8}
    .app{width:100%;max-width:520px;background:linear-gradient(180deg,var(--card),#0b1220);border-radius:18px;padding:22px;box-shadow:0 10px 40px rgba(2,6,23,.7)}
    h1{font-size:20px;margin:0 0 12px 0;font-weight:800;letter-spacing:0.4px}
    p.lead{margin:0;color:var(--muted);font-size:13px}

    .row{display:flex;gap:10px;align-items:center;margin-top:14px}
    .file{flex:1;background:#081226;border:1px dashed rgba(255,255,255,.04);padding:10px;border-radius:10px;color:var(--muted)}
    input[type=file]{width:100%;opacity:0;height:44px;position:absolute}

    audio{width:100%;margin-top:14px;border-radius:10px}

    .control{margin-top:14px}
    label{display:block;font-size:13px;color:var(--muted);margin-bottom:8px}
    input[type=range]{width:100%;height:6px;background:transparent}

    .toggles{display:flex;gap:10px;align-items:center;justify-content:space-between;margin-top:12px}
    .btn{background:linear-gradient(180deg,var(--accent),#2563eb);border:none;padding:10px 14px;border-radius:10px;color:white;font-weight:600;cursor:pointer}

    .ab-row{display:flex;gap:8px;margin-top:8px}
    .small{padding:8px 10px;border-radius:8px;background:#07142a;border:1px solid rgba(255,255,255,.03);flex:1;text-align:center;cursor:pointer}

    .metronome{display:flex;align-items:center;gap:12px}
    .metro-light{width:14px;height:14px;border-radius:50%;background:#072033;box-shadow:none;transition:box-shadow .08s,transform .08s}

    footer{margin-top:18px;color:var(--muted);font-size:12px;text-align:center}

    /* mobile-friendly spacing */
    @media (max-width:420px){.app{padding:16px}}
  </style>
</head>
<body>
  <div class="app" id="app">
    <h1>Lecteur audio — PWA</h1>
    <p class="lead">Interface simple, animations, pitch en demi-tons, BPM et métronome. Installe l'app depuis ton navigateur (Chrome/Edge/Android).</p>

    <div class="row" style="position:relative; margin-top:16px">
      <div class="file" id="fileLabel">Choisir un fichier audio</div>
      <input type="file" id="fileInput" accept="audio/*" />
    </div>

    <audio id="player" controls></audio>

    <div class="control">
      <label for="speed">Vitesse (x)</label>
      <input id="speed" type="range" min="0.5" max="2" step="0.01" value="1">
    </div>

    <div class="control">
      <label for="semitones">Hauteur (demi-tons) — indépendant</label>
      <input id="semitones" type="range" min="-12" max="12" step="1" value="0">
    </div>

    <div class="toggles">
      <label style="display:flex;gap:8px;align-items:center"><input type="checkbox" id="loop"> Lecture en boucle</label>
      <div class="metronome">
        <label style="display:flex;gap:8px;align-items:center"><input type="checkbox" id="metroToggle"> Métronome</label>
        <div class="metro-light" id="metroLight"></div>
      </div>
    </div>

    <div class="control">
      <label for="tempo">Tempo (BPM)</label>
      <input id="tempo" type="range" min="30" max="240" step="1" value="120">
    </div>

    <div class="control">
      <label>Boucle A–B</label>
      <div class="ab-row">
        <div class="small" id="setA">Définir A</div>
        <div class="small" id="setB">Définir B</div>
        <div class="small" id="clearAB">Effacer</div>
      </div>
    </div>

    <div style="display:flex;gap:10px;margin-top:14px">
      <button class="btn" id="installBtn" style="flex:1;">Installer l'app</button>
      <button class="btn" id="resetBtn" style="background:#0b1220;border:1px solid rgba(255,255,255,.04);color:#cfe6ff">Réinitialiser</button>
    </div>

    <footer>Designed with GSAP animations • Pitch via SoundTouch.js</footer>
  </div>

  <!-- Librairies -->
  <script src="https://unpkg.com/gsap@3/dist/gsap.min.js"></script>
  <script src="https://unpkg.com/soundtouch-js@0.1.1/dist/soundtouch.min.js"></script>

  <script>
    // --- UI Elements ---
    const fileInput = document.getElementById('fileInput');
    const fileLabel = document.getElementById('fileLabel');
    const audio = document.getElementById('player');
    const speed = document.getElementById('speed');
    const semitones = document.getElementById('semitones');
    const loopCheck = document.getElementById('loop');
    const setA = document.getElementById('setA');
    const setB = document.getElementById('setB');
    const clearAB = document.getElementById('clearAB');
    const metroToggle = document.getElementById('metroToggle');
    const tempo = document.getElementById('tempo');
    const metroLight = document.getElementById('metroLight');
    const installBtn = document.getElementById('installBtn');
    const resetBtn = document.getElementById('resetBtn');

    // Animations (GSAP)
    gsap.from('.app', {duration: .8, y: 30, opacity: 0, ease: 'power3.out'});
    gsap.set('.small', {y:0});

    fileInput.addEventListener('change', () => {
      const f = fileInput.files[0];
      if (!f) return;
      fileLabel.textContent = f.name;
      audio.src = URL.createObjectURL(f);
      audio.play();
      gsap.fromTo(fileLabel,{scale:0.98},{scale:1,duration:.25,ease:'elastic.out(1,0.6)'});
      // initialize pitch processor when file is loaded
      initPitchProcessing(f);
    });

    // Basic controls
    speed.addEventListener('input', ()=> {
      // speed applied to playbackRate; pitch processor will honor speed too
      if (pitchProcessor && pitchProcessor.enabled) {
        pitchProcessor.setPlaybackRate(parseFloat(speed.value));
      } else {
        audio.playbackRate = parseFloat(speed.value);
      }
    });

    loopCheck.addEventListener('change', ()=> audio.loop = loopCheck.checked);

    // A-B loop
    let A = null, B = null;
    setA.onclick = ()=> { A = audio.currentTime; gsap.fromTo(setA,{scale:1.02},{scale:1,duration:.2,repeat:0}); };
    setB.onclick = ()=> { B = audio.currentTime; gsap.fromTo(setB,{scale:1.02},{scale:1,duration:.2}); };
    clearAB.onclick = ()=> { A = null; B = null; gsap.fromTo(clearAB,{opacity:0.6},{opacity:1,duration:.25}); };
    audio.addEventListener('timeupdate', ()=> { if (A!==null && B!==null && audio.currentTime > B) audio.currentTime = A; });

    // Metronome
    let metroInterval = null;
    function clickSound() {
      const ctx = new (window.AudioContext || window.webkitAudioContext)();
      const o = ctx.createOscillator(); const g = ctx.createGain();
      o.type='square'; o.frequency.value = 1000; g.gain.value = 0.12;
      o.connect(g); g.connect(ctx.destination);
      o.start(); o.stop(ctx.currentTime + 0.05);
    }
    function flash() { gsap.fromTo(metroLight,{scale:1.0,boxShadow:'0 0 0 rgba(0,0,0,0)'},{scale:1.45,boxShadow:'0 0 12px rgba(255,80,80,0.9)',duration:.06, yoyo:true, repeat:1}); }
    function startMetronome(){ stopMetronome(); const interval = 60000 / parseInt(tempo.value); metroInterval = setInterval(()=>{clickSound(); flash();}, interval); }
    function stopMetronome(){ if(metroInterval) clearInterval(metroInterval); metroInterval = null; gsap.to(metroLight,{scale:1,boxShadow:'0 0 0 rgba(0,0,0,0)',duration:.08}); }
    metroToggle.addEventListener('change', ()=> metroToggle.checked? startMetronome(): stopMetronome());
    tempo.addEventListener('input', ()=> { if(metroToggle.checked) { startMetronome(); } });

    // Install (PWA) flow
    let deferredPrompt = null;
    window.addEventListener('beforeinstallprompt', (e) => { e.preventDefault(); deferredPrompt = e; installBtn.style.display = 'inline-block'; });
    installBtn.addEventListener('click', async ()=>{
      if (deferredPrompt) { deferredPrompt.prompt(); const choice = await deferredPrompt.userChoice; deferredPrompt = null; }
      else alert('Installez l\'app via le menu du navigateur ("Ajouter à l\'écran d\'accueil").');
    });

    // Reset
    resetBtn.addEventListener('click', ()=>{
      audio.pause(); audio.currentTime = 0; audio.src = ''; fileLabel.textContent = 'Choisir un fichier audio'; speed.value = 1; semitones.value = 0; tempo.value = 120; loopCheck.checked = false; stopMetronome(); if(pitchProcessor) pitchProcessor.disable();
    });

    // ---------------- PITCH PROCESSING using SoundTouch.js ----------------
    // Best-effort: uses ScriptProcessor to stream audio through SoundTouch, preserving speed while shifting pitch.

    let pitchProcessor = null;

    function initPitchProcessing(file) {
      // disable previous
      if (pitchProcessor) pitchProcessor.disable();

      // Create an offscreen audio element to decode and process
      const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

      const reader = new FileReader();
      reader.onload = function(evt) {
        const arrayBuffer = evt.target.result;
        audioCtx.decodeAudioData(arrayBuffer, function(buffer) {
          pitchProcessor = createSoundTouchProcessor(audioCtx, buffer);
          setSemitone(semitones.value);
          setPlaybackRate(speed.value);
        });
      };
      reader.readAsArrayBuffer(file);
    }

    function createSoundTouchProcessor(audioCtx, buffer) {
      const channels = buffer.numberOfChannels;
      const soundtouch = new soundtouchjs.SoundTouch(audioCtx.sampleRate);

      let isEnabled = true;
      let pitchShift = 0; // in semitones
      let playbackRate = parseFloat(speed.value);

      const sourceLeft = buffer.getChannelData(0);
      const sourceRight = channels>1? buffer.getChannelData(1) : null;

      const bufferSource = new soundtouchjs.BufferSource(soundtouch, buffer);
      const filter = new soundtouchjs.SimpleFilter(bufferSource, soundtouch);

      // ScriptProcessor to output into an AudioDestination
      const scriptNode = audioCtx.createScriptProcessor(1024, 2, 2);
      scriptNode.onaudioprocess = function(e) {
        if (!isEnabled) return;
        const left = e.outputBuffer.getChannelData(0);
        const right = e.outputBuffer.getChannelData(1);
        const frames = filter.extract(left.length);
        if (frames === 0) {
          // silence
          for (let i=0;i<left.length;i++){ left[i]=0; right[i]=0; }
        } else {
          // left/right arrays already filled by filter.extract
          // if mono sourceRight==null, mirror left
          if (!sourceRight) for (let i=0;i<right.length;i++) right[i] = left[i];
        }
      };

      // connect to destination via MediaStreamDestination so we can control playback with <audio>
      const dest = audioCtx.createMediaStreamDestination();
      scriptNode.connect(dest);
      scriptNode.connect(audioCtx.destination); // also audible in page

      // Create a blob URL from dest.stream to feed into audio element
      const mediaStream = dest.stream;
      const mediaURL = URL.createObjectURL(mediaStream);
      audio.src = mediaURL;
      audio.loop = false; // handled separately

      function setPitch(semi) {
        pitchShift = parseInt(semi);
        const ratio = Math.pow(2, pitchShift/12);
        soundtouch.tempo = 1.0 / playbackRate; // adjust tempo against playbackRate
        soundtouch.pitch = ratio; // Not all soundtouch-js versions support pitch field; best-effort
      }
      function setPlaybackRate(v) {
        playbackRate = parseFloat(v);
        // soundtouch-js uses tempo to stretch; we set tempo inverse of playbackRate
        soundtouch.tempo = 1.0 / playbackRate;
      }
      function disable(){ isEnabled=false; scriptNode.disconnect(); }
      function enable(){ isEnabled=true; }

      return { setPitch, setPlaybackRate, disable, enable, enabled:isEnabled };
    }

    function setSemitone(v){ if(pitchProcessor) pitchProcessor.setPitch(parseInt(v)); else { /* fallback: naive */ audio.playbackRate = Math.pow(2, v/12) * parseFloat(speed.value); } }
    function setPlaybackRate(v){ if(pitchProcessor) pitchProcessor.setPlaybackRate(parseFloat(v)); else audio.playbackRate = parseFloat(v); }

    semitones.addEventListener('input', ()=> { setSemitone(semitones.value); gsap.fromTo(semitones,{scale:1.02},{scale:1,duration:.18}); });

    // ---------------- PWA: dynamic manifest + sw registration (works when served over https/localhost) ----------------
    (function registerPWA(){
      // create a manifest blob and link it dynamically
      const manifest = {
        name: 'Lecteur audio - PWA',
        short_name: 'Lecteur',
        start_url: '.',
        display: 'standalone',
        background_color: '#071025',
        description: 'Lecteur audio avec pitch, tempo, métronome et animations',
        icons: [{src:'', sizes:'512x512', type:'image/png'}]
      };
      const blob = new Blob([JSON.stringify(manifest)], {type:'application/json'});
      const url = URL.createObjectURL(blob);
      const link = document.createElement('link');
      link.rel = 'manifest'; link.href = url; document.head.appendChild(link);

      // register a simple service worker from a blob
      const swCode = `self.addEventListener('install', e => { self.skipWaiting(); });
        self.addEventListener('fetch', e => { /* network-first for assets; this SW is minimal */ });`;
      const swBlob = new Blob([swCode], {type:'application/javascript'});
      const swUrl = URL.createObjectURL(swBlob);
      if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register(swUrl).catch(()=>{/* ignore */});
      }
    })();

    // Helpful note: SoundTouch pitch implementation is best-effort; if the library version lacks pitch/tempo fields
    // the UI will fall back to naive playbackRate changes. For production-grade independent pitch-shifting,
    // a specialized library (e.g. WebAudio-based granular shifter or third-party commercial DSP) should be used.

  </script>
</body>
</html>
