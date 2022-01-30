<script>
  const COLS = 10;
  const ROWS = 5;
  const BOMBS = 12;
  let cells = new Map();
  let remainingDefuses = BOMBS;
  let timer = 0;
  let timerId;
  let gameStarted;
  resetCells();

  function startGame(startingPosition) {
    resetTimer();
    timerId = setInterval(() => timer++, 1000);
    resetCells();
    setBombs(startingPosition);
    setNumbers();
    gameStarted = true;
  }

  function looseGame(bombExplodedPosition) {
    gameStarted = false;
    clearInterval(timerId);
    cells.set(bombExplodedPosition, {
      ...cells.get(bombExplodedPosition),
      revealed: true,
      exploded: true,
    });
    for (let [position, { value, flagged }] of cells) {
      if (value === "💣") {
        updateCell(position, { revealed: true });
      }
      if (flagged && value !== "💣") {
        updateCell(position, { wrong: true });
      }
    }
  }

  function keyToCoords(key) {
    return key.split("-").map(Number);
  }

  function coordsToKey(x, y) {
    return `${x}-${y}`;
  }

  function revealCell(position) {
    if (gameStarted === undefined) {
      startGame(position);
    }

    if (!gameStarted) {
      return;
    }

    if (cells.get(position)?.flagged) {
      return;
    }

    updateCell(position, { revealed: true, clicked: true });

    cells.get(position)?.value === 0 && revealNeighbours(position);

    updateUI();

    if (cells.get(position)?.value === "💣") {
      looseGame(position);
    }
  }

  function getNeighbours(position) {
    const [x, y] = keyToCoords(position);
    return [
      [x + 1, y],
      [x - 1, y],
      [x, y + 1],
      [x, y - 1],
      [x - 1, y - 1],
      [x + 1, y - 1],
      [x - 1, y + 1],
      [x + 1, y + 1],
    ]
      .filter(isInBounds)
      .map(([x, y]) => coordsToKey(x, y));

    function isInBounds([x, y]) {
      return 0 <= x && x < COLS && 0 <= y && y < ROWS;
    }
  }

  function revealNeighbours(position) {
    getNeighbours(position).forEach((neighbour) => {
      if (cells.get(neighbour)?.revealed) {
        return;
      }

      revealCell(neighbour);
    });
  }

  function resetCells() {
    for (let y = 0; y < ROWS; y++) {
      for (let x = 0; x < COLS; x++) {
        cells.set(coordsToKey(x, y), {});
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
        cells.get(coordsToKey(x, y)) === "💣" ||
        coordsToKey(x, y) === startingPosition
      );
      updateCell(coordsToKey(x, y), { value: "💣" });
    }
  }

  function setNumbers() {
    for (let y = 0; y < ROWS; y++) {
      for (let x = 0; x < COLS; x++) {
        let number = 0;
        if (cells.get(coordsToKey(x, y))?.value === "💣") continue;
        getNeighbours(coordsToKey(x, y)).forEach(
          (neighbour) => cells.get(neighbour)?.value === "💣" && number++
        );
        updateCell(coordsToKey(x, y), { value: number });
      }
    }
  }

  function restart() {
    remainingDefuses = BOMBS;
    gameStarted = undefined;
    resetCells();
    resetTimer();
  }

  function resetTimer() {
    clearInterval(timerId);
    timer = 0;
  }

  function updateUI() {
    cells = cells;
  }

  function defuseBomb(position) {
    if (!gameStarted) {
      return;
    }

    cells.get(position)?.flagged ? removeFlag() : remainingDefuses && addFlag();
    updateUI();

    function addFlag() {
      remainingDefuses--;
      updateCell(position, { flagged: true });
    }

    function removeFlag() {
      remainingDefuses++;
      updateCell(position, { flagged: false });
    }
  }

  function updateCell(position, values) {
    cells.set(position, { ...cells.get(position), ...values });
  }
</script>

<div>
  <div
    class="flex h-20 w-full justify-between border-2 border-gray-900 bg-gray-600 p-2"
  >
    <div
      class="flex items-center border-2 border-gray-900 font-mono text-4xl font-bold text-red-500"
    >
      <span class="mx-1">{remainingDefuses.toString().padStart(3, "0")}</span>
    </div>
    <button
      on:click={restart}
      class="border-2 border-gray-900 px-4 font-mono text-2xl font-bold"
      >Restart</button
    >
    <div
      class="flex items-center border-2 border-gray-900 font-mono text-4xl font-bold text-red-500"
    >
      <span class="mx-1">{timer.toString().padStart(3, "0")}</span>
    </div>
  </div>
  <div
    class="-mt-0.5 grid gap-0.5 border-2 border-gray-900 bg-gray-900"
    style={`
        grid-template-columns: repeat(${COLS}, 1fr);        
      `}
  >
    {#each Array.from(cells) as [position, { value, flagged, exploded, revealed, clicked, wrong }]}
      <button
        class="
        stacker relative grid aspect-square w-8 cursor-default place-items-center border-2 border-t-gray-400 border-l-gray-400 border-b-gray-700 border-r-gray-700 bg-gray-600 font-bold disabled:border-none disabled:bg-gray-800
          {value === 1 && 'text-blue-600'}
          {value === 2 && 'text-green-500'}
          {value === 3 && 'text-red-500'}
          {value === 4 && 'text-purple-900'}
          {value === 5 && 'text-amber-800'}
          {value === 6 && 'text-sky-400'}
          {value === 7 && 'text-black'}
          {value === 8 && 'text-gray-500'}
          {exploded && '!bg-red-500'}
        "
        disabled={clicked}
        on:click={() => revealCell(position)}
        on:contextmenu|preventDefault={() => defuseBomb(position)}
      >
        {@html wrong
          ? `<span class="col-start-1 row-start-1">💣</span><span class="col-start-1 row-start-1">❌</span>`
          : flagged
          ? "🚩"
          : (revealed && value) || ""}
      </button>
    {/each}
  </div>
</div>
