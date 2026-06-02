<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Sudoku</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: sans-serif; background: #f5f5f5; color: #1a1a1a; display: flex; justify-content: center; padding: 2rem 1rem; min-height: 100vh; }
    .wrap { max-width: 520px; width: 100%; }
    h1 { font-size: 28px; font-weight: 600; margin-bottom: 4px; }
    .subtitle { font-size: 14px; color: #666; margin-bottom: 1.5rem; }
    .toolbar { display: flex; gap: 8px; margin-bottom: 1rem; align-items: center; flex-wrap: wrap; }
    .toolbar select, .toolbar button {
      height: 36px; border-radius: 8px; border: 1px solid #ccc;
      background: #fff; color: #1a1a1a; font-size: 13px;
      padding: 0 12px; cursor: pointer;
    }
    .toolbar button:hover { background: #f0f0f0; }
    .timer { margin-left: auto; font-size: 13px; color: #666; font-variant-numeric: tabular-nums; }
    #board {
      display: grid; grid-template-columns: repeat(9, 1fr);
      border: 2px solid #333; border-radius: 6px; overflow: hidden;
      width: 100%; aspect-ratio: 1;
    }
    .cell {
      display: flex; align-items: center; justify-content: center;
      font-size: clamp(13px, 2vw, 20px); cursor: pointer;
      border: 0.5px solid #ccc; background: #fff; color: #1a1a1a;
      transition: background 0.1s; user-select: none;
    }
    .cell.given { font-weight: 600; cursor: default; }
    .cell.selected { background: #bcd8f7 !important; }
    .cell.highlight { background: #eef4fb; }
    .cell.error { color: #b33; }
    .cell.border-right { border-right: 2px solid #333; }
    .cell.border-bottom { border-bottom: 2px solid #333; }
    @keyframes flash { 0%,100%{ background: #fff; } 50%{ background: #d4edda; } }
    .cell.correct-flash { animation: flash 0.4s; }
    .notes {
      font-size: 8px; display: grid;
      grid-template-columns: repeat(3,1fr); grid-template-rows: repeat(3,1fr);
      width: 100%; height: 100%; padding: 1px; color: #888; line-height: 1;
    }
    .notes span { display: flex; align-items: center; justify-content: center; }
    .numpad { display: grid; grid-template-columns: repeat(9, 1fr); gap: 6px; margin-top: 1rem; }
    .numpad button {
      height: 42px; border-radius: 8px; border: 1px solid #ccc;
      background: #fff; color: #1a1a1a; font-size: 16px; font-weight: 500; cursor: pointer;
    }
    .numpad button:hover { background: #f0f0f0; }
    .status { margin-top: 1rem; font-size: 14px; color: #555; min-height: 22px; text-align: center; }
    .win-msg { color: #0a7a52; font-weight: 600; font-size: 16px; }
    button.note-on { background: #e0f0ff !important; color: #1560a8 !important; }
  </style>
</head>
<body>
<div class="wrap">
  <h1>Sudoku</h1>
  <p class="subtitle">Preencha o tabuleiro sem repetir números de 1–9 em linhas, colunas e blocos 3×3.</p>

  <div class="toolbar">
    <select id="diff">
      <option value="easy">Fácil</option>
      <option value="medium" selected>Médio</option>
      <option value="hard">Difícil</option>
    </select>
    <button id="btn-new">Novo jogo</button>
    <button id="btn-note">Notas: OFF</button>
    <button id="btn-hint">Dica</button>
    <span class="timer" id="timer">00:00</span>
  </div>

  <div id="board"></div>
  <div class="numpad" id="numpad"></div>
  <div class="status" id="status"></div>
</div>

<script>
  const boardEl  = document.getElementById('board');
  const numpadEl = document.getElementById('numpad');
  const statusEl = document.getElementById('status');
  const timerEl  = document.getElementById('timer');
  const noteBtn  = document.getElementById('btn-note');

  let solution = [], puzzle = [], userGrid = [], notes = [];
  let selected = null, noteMode = false, timerSec = 0, timerInt = null, won = false;

  /* ---- utilidades ---- */
  function shuffle(arr) {
    for (let i = arr.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
    return arr;
  }

  function isValid(grid, r, c, n) {
    if (grid[r].includes(n)) return false;
    if (grid.some(row => row[c] === n)) return false;
    const br = Math.floor(r / 3) * 3, bc = Math.floor(c / 3) * 3;
    for (let i = 0; i < 3; i++)
      for (let j = 0; j < 3; j++)
        if (grid[br + i][bc + j] === n) return false;
    return true;
  }

  function findEmpty(grid) {
    for (let r = 0; r < 9; r++)
      for (let c = 0; c < 9; c++)
        if (grid[r][c] === 0) return [r, c];
    return null;
  }

  /* ---- gerador ---- */
  function solveSudoku(grid) {
    const empty = findEmpty(grid);
    if (!empty) return true;
    const [r, c] = empty;
    for (const n of shuffle([1,2,3,4,5,6,7,8,9])) {
      if (isValid(grid, r, c, n)) {
        grid[r][c] = n;
        if (solveSudoku(grid)) return true;
        grid[r][c] = 0;
      }
    }
    return false;
  }

  function countSolutions(grid, limit = 2) {
    const empty = findEmpty(grid);
    if (!empty) return 1;
    const [r, c] = empty;
    let count = 0;
    for (let n = 1; n <= 9; n++) {
      if (isValid(grid, r, c, n)) {
        grid[r][c] = n;
        count += countSolutions(grid, limit);
        if (count >= limit) return count;
        grid[r][c] = 0;
      }
    }
    return count;
  }

  function generatePuzzle(diff) {
    const base = Array.from({ length: 9 }, () => Array(9).fill(0));
    solveSudoku(base);
    const sol = base.map(r => [...r]);
    const puz = base.map(r => [...r]);
    const removes = diff === 'easy' ? 35 : diff === 'medium' ? 46 : 54;
    let cells = shuffle([...Array(81).keys()]);
    let removed = 0;
    for (const idx of cells) {
      if (removed >= removes) break;
      const r = Math.floor(idx / 9), c = idx % 9;
      const backup = puz[r][c];
      puz[r][c] = 0;
      const test = puz.map(row => [...row]);
      if (countSolutions(test) === 1) removed++;
      else puz[r][c] = backup;
    }
    return { solution: sol, puzzle: puz };
  }

  /* ---- jogo ---- */
  function newGame() {
    won = false;
    statusEl.textContent = '';
    clearInterval(timerInt);
    timerSec = 0;
    timerEl.textContent = '00:00';
    const diff = document.getElementById('diff').value;
    const { solution: sol, puzzle: puz } = generatePuzzle(diff);
    solution = sol;
    puzzle   = puz;
    userGrid = puz.map(r => [...r]);
    notes    = Array.from({ length: 9 }, () => Array.from({ length: 9 }, () => new Set()));
    selected = null;
    timerInt = setInterval(() => {
      timerSec++;
      const m = String(Math.floor(timerSec / 60)).padStart(2, '0');
      const s = String(timerSec % 60).padStart(2, '0');
      timerEl.textContent = `${m}:${s}`;
    }, 1000);
    render();
  }

  function render() {
    boardEl.innerHTML = '';
    for (let r = 0; r < 9; r++) {
      for (let c = 0; c < 9; c++) {
        const cell = document.createElement('div');
        cell.className = 'cell';
        if (puzzle[r][c] !== 0) cell.classList.add('given');
        if (c === 2 || c === 5) cell.classList.add('border-right');
        if (r === 2 || r === 5) cell.classList.add('border-bottom');
        if (selected) {
          if (selected[0] === r && selected[1] === c) cell.classList.add('selected');
          else if (selected[0] === r || selected[1] === c ||
            (Math.floor(selected[0]/3) === Math.floor(r/3) &&
             Math.floor(selected[1]/3) === Math.floor(c/3)))
            cell.classList.add('highlight');
        }
        const val = userGrid[r][c];
        if (val !== 0) {
          cell.textContent = val;
          if (puzzle[r][c] === 0 && val !== solution[r][c])
            cell.classList.add('error');
        } else if (notes[r][c].size > 0) {
          const nd = document.createElement('div');
          nd.className = 'notes';
          for (let i = 1; i <= 9; i++) {
            const sp = document.createElement('span');
            sp.textContent = notes[r][c].has(i) ? i : '';
            nd.appendChild(sp);
          }
          cell.appendChild(nd);
        }
        cell.addEventListener('click', () => selectCell(r, c));
        boardEl.appendChild(cell);
      }
    }
  }

  function selectCell(r, c) {
    if (won) return;
    selected = [r, c];
    render();
  }

  function inputNumber(n) {
    if (!selected || won) return;
    const [r, c] = selected;
    if (puzzle[r][c] !== 0) return;
    if (noteMode) {
      if (userGrid[r][c] !== 0) return;
      notes[r][c].has(n) ? notes[r][c].delete(n) : notes[r][c].add(n);
    } else {
      notes[r][c].clear();
      userGrid[r][c] = (userGrid[r][c] === n) ? 0 : n;
      if (n !== 0 && userGrid[r][c] === solution[r][c]) {
        render();
        const idx = r * 9 + c;
        boardEl.children[idx].classList.add('correct-flash');
        setTimeout(() => boardEl.children[idx].classList.remove('correct-flash'), 400);
        checkWin();
        return;
      }
    }
    render();
  }

  function checkWin() {
    for (let r = 0; r < 9; r++)
      for (let c = 0; c < 9; c++)
        if (userGrid[r][c] !== solution[r][c]) return;
    won = true;
    clearInterval(timerInt);
    const m = String(Math.floor(timerSec / 60)).padStart(2, '0');
    const s = String(timerSec % 60).padStart(2, '0');
    statusEl.innerHTML = `<span class="win-msg">Parabéns! Concluído em ${m}:${s} 🎉</span>`;
  }

  /* ---- numpad ---- */
  for (let n = 1; n <= 9; n++) {
    const b = document.createElement('button');
    b.textContent = n;
    b.addEventListener('click', () => inputNumber(n));
    numpadEl.appendChild(b);
  }

  /* ---- controles ---- */
  document.getElementById('btn-new').addEventListener('click', newGame);

  document.getElementById('btn-hint').addEventListener('click', () => {
    if (!selected || won) return;
    const [r, c] = selected;
    if (puzzle[r][c] !== 0) return;
    notes[r][c].clear();
    userGrid[r][c] = solution[r][c];
    statusEl.textContent = 'Dica usada!';
    setTimeout(() => { if (!won) statusEl.textContent = ''; }, 1500);
    checkWin();
    render();
  });

  noteBtn.addEventListener('click', () => {
    noteMode = !noteMode;
    noteBtn.textContent = `Notas: ${noteMode ? 'ON' : 'OFF'}`;
    noteMode ? noteBtn.classList.add('note-on') : noteBtn.classList.remove('note-on');
  });

  document.addEventListener('keydown', e => {
    if (won || !selected) return;
    if (e.key >= '1' && e.key <= '9') inputNumber(parseInt(e.key));
    if (e.key === 'Backspace' || e.key === 'Delete' || e.key === '0') inputNumber(0);
    const [r, c] = selected;
    if (e.key === 'ArrowUp'    && r > 0) selectCell(r - 1, c);
    if (e.key === 'ArrowDown'  && r < 8) selectCell(r + 1, c);
    if (e.key === 'ArrowLeft'  && c > 0) selectCell(r, c - 1);
    if (e.key === 'ArrowRight' && c < 8) selectCell(r, c + 1);
  });

  newGame();
</script>
</body>
</html>
