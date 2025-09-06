<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>One-Click Voice Cloning (TTS)</title>
  <style>
    :root { --maxw: 880px; }
    body { font-family: system-ui, Arial, sans-serif; margin: 0; padding: 0; background: #f7f7fb; color: #111; }
    .wrap { max-width: var(--maxw); margin: 0 auto; padding: 32px 20px; }
    h1 { font-size: 28px; margin: 0 0 8px; }
    p.lead { margin: 0 0 18px; color: #444; }
    textarea { width: 100%; height: 140px; padding: 12px; border-radius: 14px; border: 1px solid #ddd; font-size: 16px; resize: vertical; }
    .row { display: flex; gap: 12px; align-items: center; margin-top: 14px; flex-wrap: wrap; }
    button { padding: 12px 18px; font-size: 16px; border-radius: 14px; border: 0; cursor: pointer; background: #111; color: #fff; }
    button:disabled { opacity: .6; cursor: not-allowed; }
    .hint { font-size: 13px; color: #666; margin-top: 10px; }
    .card { background: #fff; border: 1px solid #eee; border-radius: 18px; padding: 18px; margin-top: 16px; }
    details { margin-top: 12px; }
    summary { cursor: pointer; }
    .badge { display: inline-block; padding: 2px 10px; border-radius: 999px; background: #eef; color: #224; font-size: 12px; margin-left: 6px; }
    footer { margin-top: 28px; font-size: 12px; color: #666; }
    select, input[type="range"] { padding: 8px; border-radius: 10px; border: 1px solid #ddd; background: #fff; }
    label { font-size: 13px; color: #333; }
  </style>
</head>
<body>
  <div class="wrap">
    <h1>🎤 One-Click Voice Cloning <span class="badge">Browser-only</span></h1>
    <p class="lead">Text likho → ek click pe bolay. (Pure browser TTS; no server, no cost.)</p>

    <div class="card">
      <label for="textInput">Your Text</label>
      <textarea id="textInput" placeholder="Type something...">Assalam o Alaikum! Ye One-Click Voice Cloning TTS demo hai. Aap apna text likhein aur 🔊 Bolay par click karen.</textarea>
      <div class="row">
        <button id="speakBtn">🔊 Bolay (Speak)</button>
        <button id="stopBtn">⏹️ Stop</button>
        <span id="status" class="hint">Ready.</span>
      </div>

      <details>
        <summary>⚙️ Options (optional)</summary>
        <div class="row">
          <label>Voice:
            <select id="voiceSelect"></select>
          </label>
          <label>Rate:
            <input id="rate" type="range" min="0.5" max="1.5" step="0.1" value="1" />
          </label>
          <label>Pitch:
            <input id="pitch" type="range" min="0" max="2" step="0.1" value="1" />
          </label>
        </div>
        <p class="hint">Tip: Urdu ke liye list se <b>ur-PK</b> ya <b>hi-IN</b> voice select karein (agar device par installed ho).</p>
      </details>
    </div>

    <footer>
      <p>Browser Speech Synthesis API use hoti hai. Exact “voice cloning” (aapki khud ki awaaz) browser-only mode main limited hai. Ye MVP TTS ke liye best hai.</p>
    </footer>
  </div>

  <script>
    const textInput = document.getElementById('textInput');
    const speakBtn  = document.getElementById('speakBtn');
    const stopBtn   = document.getElementById('stopBtn');
    const statusEl  = document.getElementById('status');
    const voiceSel  = document.getElementById('voiceSelect');
    const rateEl    = document.getElementById('rate');
    const pitchEl   = document.getElementById('pitch');

    let voices = [];

    function populateVoices() {
      voices = speechSynthesis.getVoices();
      voiceSel.innerHTML = '';
      voices.forEach((v, i) => {
        const opt = document.createElement('option');
        opt.value = i;
        opt.textContent = v.name + ' — ' + v.lang + (v.default ? ' (default)' : '');
        voiceSel.appendChild(opt);
      });

      // Try to auto-pick an Urdu/Hindi/English voice
      const preferred = ['ur-PK', 'hi-IN', 'en-GB', 'en-US'];
      let index = voices.findIndex(v => preferred.includes(v.lang));
      if (index === -1) {
        index = voices.findIndex(v => preferred.some(p => v.lang.startsWith(p.split('-')[0])));
      }
      if (index >= 0) voiceSel.value = String(index);
    }

    populateVoices();
    window.speechSynthesis.onvoiceschanged = populateVoices;

    function speak() {
      const text = textInput.value.trim();
      if (!text) {
        alert('Please type some text.');
        return;
      }
      speechSynthesis.cancel(); // stop any previous speech

      const utter = new SpeechSynthesisUtterance(text);
      const vIndex = parseInt(voiceSel.value, 10);
      if (voices[vIndex]) {
        utter.voice = voices[vIndex];
        utter.lang = voices[vIndex].lang;
      }
      utter.rate = parseFloat(rateEl.value || '1');
      utter.pitch = parseFloat(pitchEl.value || '1');

      utter.onstart = () => statusEl.textContent = 'Speaking...';
      utter.onend   = () => statusEl.textContent = 'Done.';
      utter.onerror = (e) => { console.error(e); statusEl.textContent = 'Error speaking.'; };

      speechSynthesis.speak(utter);
    }

    function stopSpeak() {
      speechSynthesis.cancel();
      statusEl.textContent = 'Stopped.';
    }

    speakBtn.addEventListener('click', speak);
    stopBtn.addEventListener('click', stopSpeak);
  </script>
</body>
</html>
