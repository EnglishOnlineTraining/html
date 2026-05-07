# html
a basic html section for language learning and vocab


<style>
.phrase-chip {
  display: inline-block;
  padding: 8px 14px;
  margin: 5px;
  background: var(--color-background-primary);
  border: 0.5px solid var(--color-border-secondary);
  border-radius: var(--border-radius-md);
  font-size: 14px;
  color: var(--color-text-primary);
  cursor: grab;
  user-select: none;
  transition: opacity 0.15s;
}
.phrase-chip:active { cursor: grabbing; opacity: 0.5; }
.phrase-chip.dragging { opacity: 0.3; }
.phrase-chip.placed { background: #E1F5EE; border-color: #0F6E56; color: #085041; }
.drop-zone {
  min-height: 52px;
  border: 1.5px dashed var(--color-border-secondary);
  border-radius: var(--border-radius-md);
  padding: 8px;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 4px;
  transition: background 0.15s, border-color 0.15s;
  background: var(--color-background-secondary);
}
.drop-zone.over { border-color: #1D9E75; background: #E1F5EE22; }
.drop-zone.correct { border-color: #1D9E75; background: #E1F5EE; }
.drop-zone.incorrect { border-color: #E24B4A; background: #FCEBEB; }
.category-label {
  font-size: 13px;
  font-weight: 500;
  color: var(--color-text-secondary);
  margin-bottom: 6px;
}
.sentence-context {
  font-size: 14px;
  color: var(--color-text-primary);
  margin-bottom: 8px;
  line-height: 1.5;
}
.feedback-msg { font-size: 13px; margin-top: 6px; min-height: 18px; }
.feedback-msg.ok { color: #0F6E56; }
.feedback-msg.err { color: #A32D2D; }
</style>

<h2 class="sr-only">Drag and drop word bank activity: sort phrases about life priorities into the correct sentence</h2>

<div style="padding: 1rem 0;">

  <p style="font-size: 15px; color: var(--color-text-secondary); margin: 0 0 1.5rem;">Drag each phrase from the word bank into the correct sentence. Each sentence needs one phrase.</p>

  <div style="background: var(--color-background-secondary); border: 0.5px solid var(--color-border-tertiary); border-radius: var(--border-radius-lg); padding: 1rem 1.25rem; margin-bottom: 1.5rem;">
    <p style="font-size: 13px; font-weight: 500; color: var(--color-text-secondary); margin: 0 0 10px;">Word bank — drag from here</p>
    <div id="bank" style="display: flex; flex-wrap: wrap; min-height: 44px;">
    </div>
  </div>

  <div style="display: flex; flex-direction: column; gap: 1.25rem;">

    <div>
      <p class="sentence-context">1. They seem to <span style="color: var(--color-text-secondary);">[  ?  ]</span> luxury holidays — look at their boat and champagne!</p>
      <div class="drop-zone" id="zone-0" data-answer="make enough money to afford"></div>
      <p class="feedback-msg" id="fb-0"></p>
    </div>

    <div>
      <p class="sentence-context">2. People who change jobs every year might <span style="color: var(--color-text-secondary);">[  ?  ]</span> in the same position forever.</p>
      <div class="drop-zone" id="zone-1" data-answer="not want to have a boring"></div>
      <p class="feedback-msg" id="fb-1"></p>
    </div>

    <div>
      <p class="sentence-context">3. Staying in the same job for over four years suggests you <span style="color: var(--color-text-secondary);">[  ?  ]</span> about career security.</p>
      <div class="drop-zone" id="zone-2" data-answer="worry"></div>
      <p class="feedback-msg" id="fb-2"></p>
    </div>

    <div>
      <p class="sentence-context">4. It's important <span style="color: var(--color-text-secondary);">[  ?  ]</span>, even if they are very different from your own.</p>
      <div class="drop-zone" id="zone-3" data-answer="to accept other people's priorities"></div>
      <p class="feedback-msg" id="fb-3"></p>
    </div>

    <div>
      <p class="sentence-context">5. Jumping between jobs shows you want <span style="color: var(--color-text-secondary);">[  ?  ]</span> — full of new experiences and challenges.</p>
      <div class="drop-zone" id="zone-4" data-answer="an adventurous life"></div>
      <p class="feedback-msg" id="fb-4"></p>
    </div>

    <div>
      <p class="sentence-context">6. Someone focused only on earning more and more money might <span style="color: var(--color-text-secondary);">[  ?  ]</span>.</p>
      <div class="drop-zone" id="zone-5" data-answer="have the wrong priorities"></div>
      <p class="feedback-msg" id="fb-5"></p>
    </div>

  </div>

  <div style="display: flex; gap: 10px; margin-top: 1.5rem; flex-wrap: wrap;">
    <button id="check-btn" onclick="checkAll()">Check answers</button>
    <button id="reset-btn" onclick="resetAll()">Reset <i class="ti ti-refresh" aria-hidden="true"></i></button>
  </div>

  <p id="score-msg" style="font-size: 15px; font-weight: 500; margin-top: 1rem; min-height: 24px; color: var(--color-text-primary);"></p>

</div>

<script>
const phrases = [
  { id: 'p0', text: 'make enough money to afford' },
  { id: 'p1', text: 'not want to have a boring' },
  { id: 'p2', text: 'worry' },
  { id: 'p3', text: 'to accept other people\'s priorities' },
  { id: 'p4', text: 'an adventurous life' },
  { id: 'p5', text: 'have the wrong priorities' },
];

let draggedId = null;
let dragSourceZone = null;

function shuffle(arr) {
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}

function makeChip(p) {
  const chip = document.createElement('span');
  chip.className = 'phrase-chip';
  chip.id = p.id;
  chip.textContent = p.text;
  chip.draggable = true;
  chip.addEventListener('dragstart', e => {
    draggedId = p.id;
    dragSourceZone = chip.parentElement.id;
    chip.classList.add('dragging');
    e.dataTransfer.effectAllowed = 'move';
  });
  chip.addEventListener('dragend', () => chip.classList.remove('dragging'));
  return chip;
}

function setupZone(zone) {
  zone.addEventListener('dragover', e => { e.preventDefault(); zone.classList.add('over'); });
  zone.addEventListener('dragleave', () => zone.classList.remove('over'));
  zone.addEventListener('drop', e => {
    e.preventDefault();
    zone.classList.remove('over');
    if (!draggedId) return;
    const existingChip = zone.querySelector('.phrase-chip');
    const draggedChip = document.getElementById(draggedId);
    if (existingChip && existingChip.id !== draggedId) {
      const src = dragSourceZone === zone.id ? zone : (document.getElementById(dragSourceZone) || document.getElementById('bank'));
      src.appendChild(existingChip);
      existingChip.classList.remove('placed');
    }
    zone.appendChild(draggedChip);
    draggedChip.classList.add('placed');
    clearFeedback();
    draggedId = null;
  });
}

function clearFeedback() {
  document.querySelectorAll('.feedback-msg').forEach(el => { el.textContent = ''; el.className = 'feedback-msg'; });
  document.querySelectorAll('.drop-zone').forEach(el => el.classList.remove('correct', 'incorrect'));
  document.getElementById('score-msg').textContent = '';
}

function checkAll() {
  const zones = document.querySelectorAll('.drop-zone');
  let correct = 0;
  zones.forEach((zone, i) => {
    const chip = zone.querySelector('.phrase-chip');
    const fb = document.getElementById('fb-' + i);
    if (!chip) {
      fb.textContent = 'No answer placed yet.';
      fb.className = 'feedback-msg err';
      return;
    }
    if (chip.textContent.trim() === zone.dataset.answer.trim()) {
      zone.classList.add('correct');
      zone.classList.remove('incorrect');
      fb.textContent = 'Correct!';
      fb.className = 'feedback-msg ok';
      correct++;
    } else {
      zone.classList.add('incorrect');
      zone.classList.remove('correct');
      fb.textContent = 'Not quite — try again.';
      fb.className = 'feedback-msg err';
    }
  });
  const scoreEl = document.getElementById('score-msg');
  scoreEl.textContent = correct + ' / ' + zones.length + ' correct';
  if (correct === zones.length) {
    scoreEl.textContent += ' — well done!';
    scoreEl.style.color = '#0F6E56';
  } else {
    scoreEl.style.color = 'var(--color-text-primary)';
  }
}

function resetAll() {
  const bank = document.getElementById('bank');
  document.querySelectorAll('.phrase-chip').forEach(chip => {
    chip.classList.remove('placed', 'dragging');
    bank.appendChild(chip);
  });
  clearFeedback();
}

const bank = document.getElementById('bank');
const bankSetup = document.getElementById('bank');
shuffle(phrases).forEach(p => bank.appendChild(makeChip(p)));
document.querySelectorAll('.drop-zone').forEach(setupZone);

bankSetup.addEventListener('dragover', e => e.preventDefault());
bankSetup.addEventListener('drop', e => {
  e.preventDefault();
  if (!draggedId) return;
  const chip = document.getElementById(draggedId);
  chip.classList.remove('placed');
  bank.appendChild(chip);
  clearFeedback();
  draggedId = null;
});
</script>
