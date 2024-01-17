<script lang="ts">
  const WIDTH = 25;
  const HEIGHT = 25;
  const MINES = WIDTH * HEIGHT * 0.1;

  const mines: boolean[] = Array(WIDTH * HEIGHT).fill(false);
  let endGame = false;

  let generatedMines = 0;
  while (generatedMines < MINES) {
    const row = Math.floor(Math.random() * HEIGHT);
    const col = Math.floor(Math.random() * WIDTH);
    const i = row * WIDTH + col;
    if (!mines[i]) {
      mines[i] = true;
      generatedMines++;
    }
  }

  let boardReveals: boolean[][] = Array(HEIGHT)
    .fill(undefined)
    .map(() => Array(WIDTH).fill(false));

  const cellNumbers: number[] = Array(HEIGHT * WIDTH).fill(-1);
  const revealCell = (row: number, col: number) => {
    if (row < 0 || col < 0 || row >= HEIGHT || col >= WIDTH) return;
    if (boardReveals[row][col]) return;

    boardReveals = boardReveals.map((rowCells, _row) =>
      rowCells.map((revealedCell, _col) =>
        _row === row && _col === col ? true : revealedCell
      )
    );

    if (mines[row * WIDTH + col]) endGame = true;

    let hits = 0;
    for (let r = Math.max(0, row - 1); r <= Math.min(HEIGHT, row + 1); r++) {
      for (let c = Math.max(0, col - 1); c <= Math.min(WIDTH, col + 1); c++) {
        hits += mines[r * WIDTH + c] ? 1 : 0;
      }
    }
    cellNumbers[row * WIDTH + col] = hits;
    if (hits === 0) {
      revealCell(row - 1, col - 1);
      revealCell(row, col - 1);
      revealCell(row + 1, col - 1);
      revealCell(row - 1, col);
      // revealCell(row, col);
      revealCell(row + 1, col);
      revealCell(row - 1, col + 1);
      revealCell(row, col + 1);
      revealCell(row + 1, col + 1);
    }
  };

  const flags: boolean[] = Array(HEIGHT * WIDTH).fill(false);
</script>

<main class:endGame>
  {#each boardReveals as boardRow, row (row)}
    <div class="row">
      {#each boardRow as cellIsRevealed, col (col)}
        {#if cellIsRevealed || endGame}
          {#if mines[row * WIDTH + col]}
            <div class="square revealed">*</div>
          {:else}
            <div class="square revealed">
              {cellNumbers[row * WIDTH + col] || ""}
            </div>
          {/if}
        {:else}
          <!-- svelte-ignore a11y-no-static-element-interactions -->
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          {@const isFlagged = flags[row * WIDTH + col]}
          <div
            class="square"
            on:click={(e) => {
              if (e.shiftKey) flags[row * WIDTH + col] = !isFlagged;
              else if (!isFlagged) revealCell(row, col);
            }}
          >
            {#if isFlagged}?{/if}
          </div>
        {/if}
      {/each}
    </div>
  {/each}
</main>

<style>
  .endGame {
    filter: opacity(0.5);
    pointer-events: none;
  }

  .row {
    display: flex;
    flex-direction: row;
    user-select: none;
  }
  .square {
    --size: 20px;

    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    height: var(--size);
    width: var(--size);
    background-color: rgba(0, 0, 0, 0.25);
    border: 4px solid black;
    border-color: rgba(255, 255, 255, 0.5) rgba(0, 0, 0, 0.5) rgba(0, 0, 0, 0.5)
      rgba(255, 255, 255, 0.5);
    cursor: pointer;
  }
  .square.revealed {
    background-color: rgba(0, 0, 0, 0.125);
    border-color: rgba(0, 0, 0, 0.1);
    cursor: initial;
  }
</style>
