<script>
  const COLS = 10;
  const ROWS = 5;
  let bombs = 10;
  let startingBombs;
  let cells = new Map();
  let timer = 0;
  let timerId;
  let gameStarted;
  let gameWon;
  let rightClicking;
  let rightClickStarted;
  const IS_MOBILE = isMobile();
  resetCells();

  function startGame(startingPosition) {
    startingBombs = bombs;
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
    updateCell(bombExplodedPosition, { revealed: true, exploded: true });
    for (let [position, { value }] of cells) {
      if (value === "💣") {
        updateCell(position, { revealed: true });
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

    checkGameWon();
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
      .map(([x, y]) => coordsToKey(x, y))
      .filter(isInBounds);

    function isInBounds(position) {
      return cells.has(position);
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
    [...cells]
      .map(([position]) => position)
      .filter((position) => position !== startingPosition)
      .sort(randomize)
      .slice(0, bombs)
      .forEach(addBomb);

    function randomize() {
      return Math.random() > 0.5 ? 1 : -1;
    }

    function addBomb(position) {
      updateCell(position, { value: "💣" });
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
    gameWon = false;
    bombs = startingBombs;
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

    if (
      !IS_MOBILE &&
      cells.get(position).lastRightClick === rightClickStarted
    ) {
      return;
    }

    updateCell(position, { lastRightClick: rightClickStarted });

    cells.get(position)?.flagged ? removeFlag() : bombs && addFlag();
    updateUI();

    function addFlag() {
      bombs--;
      updateCell(position, { flagged: true });
      checkGameWon();
    }

    function removeFlag() {
      bombs++;
      updateCell(position, { flagged: false });
    }
  }

  function updateCell(position, values) {
    cells.set(position, { ...cells.get(position), ...values });
  }

  function checkGameWon() {
    let winConditions = true;
    for (let [_position, { revealed, flagged, value }] of cells) {
      if (flagged && value === "💣") continue; // bomb correctly guessed
      if (revealed) continue; // normal cell

      winConditions = false;
      break;
    }

    winConditions && winGame();
  }

  function winGame() {
    gameWon = true;
    gameStarted = false;
    clearInterval(timerId);
    for (let [position] of cells) {
      updateCell(position, { revealed: true });
    }
  }

  function isMobile() {
    let check = false;
    (function (a) {
      if (
        /(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino/i.test(
          a
        ) ||
        /1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(
          a.substr(0, 4)
        )
      )
        check = true;
      // @ts-ignore
    })(navigator.userAgent || navigator.vendor || window.opera);
    return check;
  }

  function handleDragOn(e) {
    if (IS_MOBILE) return;
    if (e.button === 2) {
      rightClicking = true;
      rightClickStarted = Date.now();
    }
  }

  function handleDragOff(e) {
    if (IS_MOBILE) return;
    e.button === 2 && (rightClicking = false);
  }

  function handleDrag(position) {
    if (IS_MOBILE) return;
    rightClicking && defuseBomb(position);
  }
</script>

<div class="relative select-none">
  {#if gameWon}
    <div class="absolute inset-x-0 bottom-full mx-auto mb-2 w-max text-4xl">
      You win
    </div>
  {/if}
  <div
    class="flex h-20 w-full justify-between border-2 border-gray-900 bg-gray-600 p-2"
  >
    <div class="flex items-center border-2 border-gray-900">
      <input
        type="number"
        class="mx-1 inline-block w-[3ch] bg-transparent font-mono text-4xl font-bold text-red-500"
        disabled={gameStarted}
        min="0"
        max={ROWS * COLS - 1}
        bind:value={bombs}
      />
    </div>
    <button
      on:click={restart}
      class="border-2 border-gray-900 px-4 font-mono text-2xl font-bold"
      >Restart</button
    >
    <div class="flex items-center border-2 border-gray-900">
      <span class="mx-1 font-mono text-4xl font-bold text-red-500"
        >{timer.toString().padStart(3, "0")}</span
      >
    </div>
  </div>
  <div
    class="-mt-0.5 grid gap-0.5 border-2 border-gray-900 bg-gray-900"
    style={`
        grid-template-columns: repeat(${COLS}, 1fr);        
      `}
    on:mousedown={handleDragOn}
    on:mouseup={handleDragOff}
  >
    {#each Array.from(cells) as [position, { value, flagged, exploded, revealed, clicked }]}
      <button
        data-position={position}
        class="
        relative grid aspect-square w-8 cursor-default place-items-center border-2 border-t-gray-400 border-l-gray-400 border-b-gray-700 border-r-gray-700 bg-gray-600 font-bold disabled:border-none disabled:bg-gray-800
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
        on:mousemove={() => handleDrag(position)}
      >
        {@html !gameStarted && flagged && value !== "💣" // wrong bomb guess
          ? `<span class="col-start-1 row-start-1">💣</span><span class="col-start-1 row-start-1">❌</span>`
          : flagged
          ? "🚩"
          : (revealed && value) || ""}
      </button>
    {/each}
  </div>
</div>
