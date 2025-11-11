# webova-stranka-hudba-ritual-a-poezie-v-egypte
webová stránka na téma Hudba, rituál a poezie v Egyptē
// script.js — interaktivita a přístupnost
document.addEventListener('DOMContentLoaded', () => {
  // Controls for contrast and text size
  const html = document.documentElement;
  const contrastBtn = document.getElementById('toggle-contrast');
  const textsizeBtn = document.getElementById('toggle-textsize');

  contrastBtn.addEventListener('click', () => {
    const on = html.classList.toggle('high-contrast');
    contrastBtn.setAttribute('aria-pressed', String(on));
  });
  textsizeBtn.addEventListener('click', () => {
    const on = html.classList.toggle('large-text');
    textsizeBtn.setAttribute('aria-pressed', String(on));
  });

  // Timeline interaction
  const timeRange = document.getElementById('time-range');
  const yearOutput = document.getElementById('year-output');
  const timelineText = document.getElementById('timeline-text');
  const timelineHeading = document.getElementById('timeline-heading');

  function describeYear(y){
    // Simple illustrative milestones
    if(y < -2000) return {title:'Stará říše a rituály', text:'Hudba spojená s chrámy, jednoduché rytmické nástroje a hlas.'};
    if(y < -1000) return {title:'Střední doba', text:'Rozvoj hudebních nástrojů a notací v nástinu. Poezie jako součást obřadů.'};
    if(y < 0) return {title:'Pozdní antika', text:'Vlivy Blízkého východu a řecké tradice.'};
    if(y < 1800) return {title:'Středověk a novověk', text:'Lidové písně a rituální formy přetrvávají.'};
    if(y < 1950) return {title:'20. století', text:'Nahrávky a moderní interpretace tradičních rituálů.'};
    return {title:'Současnost', text:'Experimenty mezi tradičními formami a moderní hudbou.'};
  }

  function updateTimeline(){
    const year = Number(timeRange.value);
    yearOutput.value = year;
    yearOutput.textContent = year;
    const desc = describeYear(year);
    timelineHeading.textContent = desc.title;
    timelineText.textContent = desc.text;
  }

  timeRange.addEventListener('input', updateTimeline);
  updateTimeline();

  // Poem reader with SpeechSynthesis (fallback if supported)
  const poemButtons = Array.from(document.querySelectorAll('.poem-item'));
  const poemReader = document.getElementById('poem-reader');
  const poemTitle = document.getElementById('poem-title');
  const poemText = document.getElementById('poem-text');
  const playPoemBtn = document.getElementById('play-poem');
  const stopPoemBtn = document.getElementById('stop-poem');
  const highlightCheckbox = document.getElementById('highlight-lines');
  let utterance = null;

  const poems = {
    "Převod staré hymny": [
      "Ó Nil, tvé proudy hladí zemi,",
      "staré písně v srdci znějí.",
      "Bubny volají — rituál pokračuje."
    ],
    "Moderní poezie: Nil": [
      "Nil proudí, mění tvář,",
      "města rostou, písek spí.",
      "Hlasy paměti kmitají v noci.",
      "Poezie jako most mezi dny."
    ]
  };

  poemButtons.forEach(btn=>{
    btn.addEventListener('click', ()=>{
      const title = btn.getAttribute('data-title');
      const lines = poems[title] || ["(Obsah není dostupný)"];
      poemTitle.textContent = title;
      poemText.textContent = lines.join('\n\n');
      poemReader.hidden = false;
      poemReader.focus();
      playPoemBtn.disabled = false;
    });
  });

  function speakPoem(){
    if(!('speechSynthesis' in window)){ alert('Váš prohlížeč nepodporuje hlasovou syntézu.'); return; }
    const text = poemText.textContent;
    utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = 'cs-CZ';
    // optional: slow down for clarity
    utterance.rate = 0.95;
    if(highlightCheckbox.checked){
      // highlight simulation: step through lines
      const lines = text.split('\\n\\n');
      let idx = 0;
      utterance.onboundary = (e) => {
        // best-effort: not all browsers provide precise boundary info
      };
      const step = () => {
        poemText.innerHTML = lines.map((l,i)=> i===idx? '<mark>'+escapeHtml(l)+'</mark>':escapeHtml(l)).join('\\n\\n');
        idx++;
        if(idx < lines.length){
          setTimeout(step, 800);
        }
      };
      step();
    }
    speechSynthesis.speak(utterance);
    playPoemBtn.disabled = true;
    stopPoemBtn.disabled = false;
  }

  function stopPoem(){
    if(utterance) speechSynthesis.cancel();
    playPoemBtn.disabled = false;
    stopPoemBtn.disabled = true;
  }

  function escapeHtml(s){ return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }

  playPoemBtn.addEventListener('click', speakPoem);
  stopPoemBtn.addEventListener('click', stopPoem);

  // Rhythm player using Web Audio API
  const audioCtx = (window.AudioContext || window.webkitAudioContext) ? new (window.AudioContext || window.webkitAudioContext)() : null;
  let rhythmInterval = null;

  function playPattern(patternStr){
    if(!audioCtx){ alert('AudioContext není dostupný v tomto prohlížeči.'); return; }
    const pattern = patternStr.split(',').map(s=>Number(s.trim()));
    const bpm = Number(document.getElementById('bpm').value) || 90;
    const beatMs = (60000 / bpm);
    let i=0;
    rhythmInterval = setInterval(()=>{
      const dur = pattern[i % pattern.length];
      const now = audioCtx.currentTime;
      if(dur>0){
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = 'square';
        osc.frequency.value = 200;
        gain.gain.value = 0.05;
        osc.connect(gain).connect(audioCtx.destination);
        osc.start(now);
        osc.stop(now + (dur/1000) * 0.6);
      }
      i++;
    }, beatMs);
  }

  function stopPattern(){
    if(rhythmInterval) { clearInterval(rhythmInterval); rhythmInterval = null; }
  }

  document.querySelectorAll('.play-rhythm').forEach(btn=>{
    btn.addEventListener('click', ()=> {
      const pattern = btn.getAttribute('data-pattern');
      if(!rhythmInterval) playPattern(pattern);
      else stopPattern();
      btn.textContent = rhythmInterval ? 'Zastavit rytmus' : 'Přehrát rytmus';
    });
  });

  // Synth controls
  const startSynth = document.getElementById('start-synth');
  const stopSynth = document.getElementById('stop-synth');
  let synthOsc = null;
  let synthGain = null;
  let synthTimer = null;

  startSynth.addEventListener('click', ()=>{
    if(!audioCtx){ alert('AudioContext není dostupný.'); return; }
    const freq = Number(document.getElementById('freq').value);
    const wave = document.getElementById('wave').value;
    const bpm = Number(document.getElementById('bpm').value) || 90;

    synthOsc = audioCtx.createOscillator();
    synthGain = audioCtx.createGain();
    synthOsc.type = wave;
    synthOsc.frequency.value = freq;
    synthGain.gain.value = 0.0;
    synthOsc.connect(synthGain).connect(audioCtx.destination);
    synthOsc.start();

    // simple rhythmic gate to make a pattern
    let on = true;
    synthTimer = setInterval(()=> {
      synthGain.gain.cancelScheduledValues(audioCtx.currentTime);
      synthGain.gain.setValueAtTime(on ? 0.12 : 0.0, audioCtx.currentTime);
      on = !on;
    }, (60000 / bpm) / 2);

    startSynth.disabled = true;
    stopSynth.disabled = false;
  });

  stopSynth.addEventListener('click', ()=>{
    if(synthTimer) clearInterval(synthTimer);
    if(synthOsc) { synthOsc.stop(); synthOsc.disconnect(); synthOsc = null; }
    if(synthGain) { synthGain.disconnect(); synthGain = null; }
    startSynth.disabled = false;
    stopSynth.disabled = true;
  });

  // Basic keyboard accessibility: make cards actionable with Enter
  document.querySelectorAll('.card[tabindex]').forEach(card=>{
    card.addEventListener('keydown', (e)=>{
      if(e.key === 'Enter' || e.key === ' '){ 
        const btn = card.querySelector('.play-rhythm');
        if(btn) btn.click();
        e.preventDefault();
      }
    });
  });

  // Back to top link behaviour: smooth but respects reduced motion
  document.getElementById('back-to-top').addEventListener('click', (e)=>{
    e.preventDefault();
    if(window.matchMedia('(prefers-reduced-motion: reduce)').matches){
      window.scrollTo(0,0);
    } else {
      window.scrollTo({top:0,behavior:'smooth'});
    }
  });

});
