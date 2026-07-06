<style>
@keyframes catPop { 0% { transform: scale(0.85); } 60% { transform: scale(1.06); } 100% { transform: scale(1); } }
@keyframes confettiBounce { 0%, 100% { transform: translateY(0) rotate(0deg); } 50% { transform: translateY(-10px) rotate(12deg); } }
.cmm-card { padding: 0; aspect-ratio: 1; display: flex; align-items: center; justify-content: center; cursor: pointer; transition: transform 0.15s ease, background 0.2s ease, border-color 0.2s ease; }
.cmm-card:active { transform: scale(0.96); }
.cmm-card svg { width: 68%; height: 68%; animation: catPop 0.25s ease; }
.cmm-confetti { width: 14px; height: 14px; border-radius: 4px; animation: confettiBounce 0.9s ease-in-out infinite; }
</style>
<div style="max-width: 480px; margin: 0 auto;">
<h2 class="sr-only">Find the correct pairs of cars, they have something to say!</h2>
<div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; gap: 8px; flex-wrap: wrap;">
<div>
<div style="font-size: 15px; font-weight: 500;">Cat memory match</div>
<div style="font-size: 13px; color: var(--text-secondary);">Find all 8 pairs</div>
</div>
<div style="display: flex; align-items: center; gap: 18px;">
<div style="text-align: center;">
<div style="font-size: 11px; color: var(--text-muted);">moves</div>
<div id="cmm-moves" style="font-size: 16px; font-weight: 500;">0</div>
</div>
<div style="text-align: center;">
<div style="font-size: 11px; color: var(--text-muted);">time</div>
<div id="cmm-timer" style="font-size: 16px; font-weight: 500;">0:00</div>
</div>
<button id="cmm-restart" aria-label="Restart game" style="width: 36px; height: 36px; padding: 0; display: flex; align-items: center; justify-content: center;">
<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12a9 9 0 1 1-2.64-6.36"/><path d="M21 3v6h-6"/></svg>
</button>
</div>
</div>
<div id="cmm-grid" style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px;"></div>
<div id="cmm-modal" style="display: none; min-height: 340px; background: rgba(0,0,0,0.45); border-radius: 12px; align-items: center; justify-content: center; padding: 2rem 1rem; margin-top: 1rem;">
<div style="background: var(--surface-2); border: 0.5px solid var(--border); border-radius: 12px; padding: 2rem; max-width: 320px; text-align: center;">
<div style="display: flex; justify-content: center; gap: 6px; margin-bottom: 16px;">
<div class="cmm-confetti" style="background: #D85A30; animation-delay: 0s;"></div>
<div class="cmm-confetti" style="background: #1D9E75; animation-delay: 0.1s;"></div>
<div class="cmm-confetti" style="background: #EF9F27; animation-delay: 0.2s;"></div>
<div class="cmm-confetti" style="background: #D4537E; animation-delay: 0.3s;"></div>
<div class="cmm-confetti" style="background: #378ADD; animation-delay: 0.4s;"></div>
<div class="cmm-confetti" style="background: #7F77DD; animation-delay: 0.5s;"></div>
</div>
<div style="font-size: 22px; font-weight: 500; margin-bottom: 6px;">Happy birthday Madam Ji!</div>
<div style="font-size: 14px; color: var(--text-secondary); margin-bottom: 18px;">All 8 cats found in <span id="cmm-final-moves"></span> moves and <span id="cmm-final-time"></span>.</div>
<button id="cmm-again" style="width: 100%;">Play again</button>
</div>
</div>
</div>
<script>
(function() {
  var cats = [
    { name: 'Ginger', color: '#D85A30', mark: 'plain' },
    { name: 'Smoke', color: '#5F5E5A', mark: 'plain' },
    { name: 'Midnight', color: '#2C2C2A', mark: 'plain' },
    { name: 'Snowball', color: '#FBF8F1', mark: 'outline' },
    { name: 'Blue Russian', color: '#378ADD', mark: 'plain' },
    { name: 'Patch', color: '#D4537E', mark: 'spots' },
    { name: 'Tabby', color: '#BA7517', mark: 'stripes' },
    { name: 'Tuxedo', color: '#26215C', mark: 'bib' }
  ];

  function catSVG(cat) {
    var extra = '';
    var headStroke = cat.mark === 'outline' ? ' stroke="rgba(0,0,0,0.25)" stroke-width="2"' : '';
    if (cat.mark === 'spots') {
      extra += '<ellipse cx="36" cy="46" rx="7" ry="6" fill="#F0997B" opacity="0.85"/>';
      extra += '<ellipse cx="60" cy="70" rx="6" ry="5" fill="#888780" opacity="0.85"/>';
      extra += '<ellipse cx="66" cy="42" rx="5" ry="5" fill="#F0997B" opacity="0.85"/>';
    }
    if (cat.mark === 'stripes') {
      extra += '<path d="M24 40 Q35 34 30 46" stroke="#633806" stroke-width="3" fill="none" stroke-linecap="round"/>';
      extra += '<path d="M76 40 Q65 34 70 46" stroke="#633806" stroke-width="3" fill="none" stroke-linecap="round"/>';
      extra += '<path d="M50 30 L50 42" stroke="#633806" stroke-width="3" fill="none" stroke-linecap="round"/>';
    }
    if (cat.mark === 'bib') {
      extra += '<ellipse cx="50" cy="78" rx="16" ry="12" fill="#FBF8F1"/>';
    }
    return '<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">' +
      '<polygon points="18,42 30,10 42,40" fill="' + cat.color + '"/>' +
      '<polygon points="82,42 70,10 58,40" fill="' + cat.color + '"/>' +
      '<polygon points="22,37 29,20 34,36" fill="rgba(255,255,255,0.35)"/>' +
      '<polygon points="78,37 71,20 66,36" fill="rgba(255,255,255,0.35)"/>' +
      '<ellipse cx="50" cy="58" rx="34" ry="30" fill="' + cat.color + '"' + headStroke + '/>' +
      extra +
      '<ellipse cx="38" cy="56" rx="5" ry="7" fill="#222"/>' +
      '<ellipse cx="62" cy="56" rx="5" ry="7" fill="#222"/>' +
      '<circle cx="40" cy="53" r="1.6" fill="#fff"/>' +
      '<circle cx="64" cy="53" r="1.6" fill="#fff"/>' +
      '<polygon points="46,65 54,65 50,71" fill="#F0997B"/>' +
      '<path d="M50 71 Q45 77 39 73" stroke="rgba(0,0,0,0.35)" stroke-width="1.5" fill="none" stroke-linecap="round"/>' +
      '<path d="M50 71 Q55 77 61 73" stroke="rgba(0,0,0,0.35)" stroke-width="1.5" fill="none" stroke-linecap="round"/>' +
      '<line x1="22" y1="62" x2="4" y2="57" stroke="rgba(0,0,0,0.3)" stroke-width="1.5"/>' +
      '<line x1="22" y1="67" x2="4" y2="70" stroke="rgba(0,0,0,0.3)" stroke-width="1.5"/>' +
      '<line x1="78" y1="62" x2="96" y2="57" stroke="rgba(0,0,0,0.3)" stroke-width="1.5"/>' +
      '<line x1="78" y1="67" x2="96" y2="70" stroke="rgba(0,0,0,0.3)" stroke-width="1.5"/>' +
      '</svg>';
  }

  function pawSVG() {
    return '<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" style="opacity:0.4">' +
      '<ellipse cx="50" cy="66" rx="20" ry="16" fill="var(--text-muted)"/>' +
      '<ellipse cx="28" cy="38" rx="9" ry="11" transform="rotate(-18 28 38)" fill="var(--text-muted)"/>' +
      '<ellipse cx="50" cy="28" rx="9" ry="12" fill="var(--text-muted)"/>' +
      '<ellipse cx="72" cy="38" rx="9" ry="11" transform="rotate(18 72 38)" fill="var(--text-muted)"/>' +
      '</svg>';
  }

  var gridEl = document.getElementById('cmm-grid');
  var movesEl = document.getElementById('cmm-moves');
  var timerEl = document.getElementById('cmm-timer');
  var modalEl = document.getElementById('cmm-modal');
  var finalMovesEl = document.getElementById('cmm-final-moves');
  var finalTimeEl = document.getElementById('cmm-final-time');

  var board = [];
  var cardEls = [];
  var flipped = [];
  var locked = false;
  var moves = 0;
  var matchedPairs = 0;
  var timerInterval = null;
  var timerStarted = false;
  var startTime = 0;

  function formatTime(s) {
    var m = Math.floor(s / 60);
    var sec = s % 60;
    return m + ':' + (sec < 10 ? '0' : '') + sec;
  }

  function startTimer() {
    timerStarted = true;
    startTime = Date.now();
    timerInterval = setInterval(function() {
      timerEl.textContent = formatTime(Math.floor((Date.now() - startTime) / 1000));
    }, 1000);
  }

  function shuffle(arr) {
    for (var i = arr.length - 1; i > 0; i--) {
      var j = Math.floor(Math.random() * (i + 1));
      var t = arr[i]; arr[i] = arr[j]; arr[j] = t;
    }
    return arr;
  }

  function updateCardUI(i) {
    var b = board[i];
    var el = cardEls[i];
    if (b.revealed || b.matched) {
      el.innerHTML = catSVG(cats[b.pairId]);
      el.style.background = b.matched ? 'var(--bg-success)' : 'var(--surface-2)';
      el.style.borderColor = b.matched ? 'var(--border-success)' : 'var(--border-strong)';
      el.setAttribute('aria-label', cats[b.pairId].name + (b.matched ? ', matched' : ''));
    } else {
      el.innerHTML = pawSVG();
      el.style.background = 'var(--surface-1)';
      el.style.borderColor = 'var(--border)';
      el.setAttribute('aria-label', 'Face-down card');
    }
  }

  function onCardClick(i) {
    if (locked) return;
    var b = board[i];
    if (b.revealed || b.matched) return;
    if (!timerStarted) startTimer();
    b.revealed = true;
    updateCardUI(i);
    flipped.push(i);
    if (flipped.length === 2) {
      moves++;
      movesEl.textContent = moves;
      locked = true;
      var a = flipped[0], c = flipped[1];
      if (board[a].pairId === board[c].pairId) {
        board[a].matched = true;
        board[c].matched = true;
        updateCardUI(a);
        updateCardUI(c);
        flipped = [];
        locked = false;
        matchedPairs++;
        if (matchedPairs === cats.length) {
          setTimeout(showWin, 400);
        }
      } else {
        setTimeout(function() {
          board[a].revealed = false;
          board[c].revealed = false;
          updateCardUI(a);
          updateCardUI(c);
          flipped = [];
          locked = false;
        }, 700);
      }
    }
  }

  function showWin() {
    clearInterval(timerInterval);
    finalMovesEl.textContent = moves;
    finalTimeEl.textContent = timerEl.textContent;
    modalEl.style.display = 'flex';
  }

  function initBoard() {
    clearInterval(timerInterval);
    timerStarted = false;
    moves = 0;
    matchedPairs = 0;
    flipped = [];
    locked = false;
    movesEl.textContent = '0';
    timerEl.textContent = '0:00';
    modalEl.style.display = 'none';

    var ids = [];
    for (var i = 0; i < cats.length; i++) { ids.push(i); ids.push(i); }
    shuffle(ids);
    board = ids.map(function(id) { return { pairId: id, revealed: false, matched: false }; });

    gridEl.innerHTML = '';
    cardEls = [];
    board.forEach(function(_, i) {
      var btn = document.createElement('button');
      btn.className = 'cmm-card';
      btn.style.border = '0.5px solid var(--border)';
      btn.style.borderRadius = 'var(--radius)';
      btn.style.background = 'var(--surface-1)';
      btn.addEventListener('click', function() { onCardClick(i); });
      gridEl.appendChild(btn);
      cardEls.push(btn);
      updateCardUI(i);
    });
  }

  document.getElementById('cmm-restart').addEventListener('click', initBoard);
  document.getElementById('cmm-again').addEventListener('click', initBoard);

  initBoard();
})();
</script>
