<script>
  let COLS = 10;
  let ROWS = 5;
  let cells = new Map();
  let revealedCells = new Map();
  let remainingDefuses = 12;
  let timer = 0;
  let timerId;
  let gameStarted = false;
  let buttonsGrid;
  resetCells();

  function initGame(startingPosition) {
    resetTimer();
    timerId = setInterval(() => timer++, 1000);
    resetCells();
    setBombs(startingPosition);
    setNumbers();
    gameStarted = true;
  }

  function looseGame() {
    clearInterval(timerId);
  }

  function revealCell(position, e) {
    if (revealedCells.get(position) === "🚩") {
      return;
    }

    if (!gameStarted) {
      initGame(position);
    }

    e.target.disabled = true;

    revealedCells.set(position, cells.get(position));
    updateUI();

    if (revealedCells.get(position) === "💣") {
      looseGame();
    }
  }

  function resetCells() {
    buttonsGrid &&
      [...buttonsGrid.children].forEach((child) => (child.disabled = false));
    for (let y = 0; y < ROWS; y++) {
      for (let x = 0; x < COLS; x++) {
        cells.set(`${x}-${y}`, null);
        revealedCells.set(`${x}-${y}`, null);
      }
    }
    updateUI();
  }

  function setBombs(startingPosition) {
    for (let i = 0; i < remainingDefuses; i++) {
      let x, y;
      do {
        y = Math.trunc(Math.random() * ROWS);
        x = Math.trunc(Math.random() * COLS);
      } while (
        cells.get(`${x}-${y}`) === "💣" ||
        `${x}-${y}` === startingPosition
      );
      cells.set(`${x}-${y}`, "💣");
    }
    updateUI();
  }

  function setNumbers() {
    for (let y = 0; y < ROWS; y++) {
      for (let x = 0; x < COLS; x++) {
        let number = 0;
        if (cells.get(`${x}-${y}`) === "💣") continue;
        if (cells.get(`${x + 1}-${y}`) === "💣") number++;
        if (cells.get(`${x - 1}-${y}`) === "💣") number++;
        if (cells.get(`${x}-${y + 1}`) === "💣") number++;
        if (cells.get(`${x}-${y - 1}`) === "💣") number++;
        if (cells.get(`${x - 1}-${y - 1}`) === "💣") number++;
        if (cells.get(`${x + 1}-${y - 1}`) === "💣") number++;
        if (cells.get(`${x - 1}-${y + 1}`) === "💣") number++;
        if (cells.get(`${x + 1}-${y + 1}`) === "💣") number++;
        cells.set(`${x}-${y}`, number);
      }
    }
    updateUI();
  }

  function restart() {
    gameStarted = false;
    resetCells();
    resetTimer();
  }

  function resetTimer() {
    clearInterval(timerId);
    timer = 0;
  }

  function updateUI() {
    revealedCells = revealedCells;
  }

  function defuseBomb(position) {
    function addFlag() {
      remainingDefuses--;
      revealedCells.set(position, "🚩");
    }
    function removeFlag() {
      remainingDefuses++;
      revealedCells.set(position, null);
    }
    const alreadyFlagged = revealedCells.get(position) === "🚩";
    alreadyFlagged ? removeFlag() : remainingDefuses && addFlag();
    updateUI();
  }
</script>

<div>
  <div
    class="flex h-20 w-full justify-between bg-gray-600 p-2 ring-2 ring-gray-800"
  >
    <div
      class="flex items-center font-mono text-4xl font-bold text-red-500 ring-2 ring-gray-800"
    >
      <span class="mx-1">{remainingDefuses.toString().padStart(3, "0")}</span>
    </div>
    <button
      on:click={restart}
      class="px-4 font-mono text-2xl font-bold ring-2 ring-gray-800"
      >Restart</button
    >
    <div
      class="flex items-center font-mono text-4xl font-bold text-red-500 ring-2 ring-gray-800"
    >
      <span class="mx-1">{timer.toString().padStart(3, "0")}</span>
    </div>
  </div>
  <div
    bind:this={buttonsGrid}
    class="mt-0.5 grid"
    style={`
        grid-template-columns: repeat(${COLS}, 1fr);        
      `}
  >
    {#each Array.from(revealedCells) as [position, value]}
      <button
        class="
          aspect-square w-8 cursor-default bg-gray-600 font-bold ring-2 ring-gray-800 disabled:bg-gray-700
          {value === 1 && 'text-blue-600'}
          {value === 2 && 'text-green-500'}
          {value === 3 && 'text-red-500'}
          {value === 4 && 'text-purple-900'}
          {value === 5 && 'text-amber-800'}
          {value === 6 && 'text-sky-400'}
          {value === 7 && 'text-black'}
          {value === 8 && 'text-gray-500'}
        "
        on:click={(e) => revealCell(position, e)}
        on:contextmenu|preventDefault={() => defuseBomb(position)}
      >
        {value ? value : ""}
      </button>
    {/each}
  </div>
</div>
