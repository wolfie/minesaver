<script lang="ts">
  const MINES = 5;
  const WIDTH = 5;
  const HEIGHT = 5;

  let mines: [row: number, col: number][] = [];
  let endGame = false;

  console.log();
  while (mines.length < MINES) {
    const row = Math.floor(Math.random() * HEIGHT);
    const col = Math.floor(Math.random() * WIDTH);
    if (!mines.some(([_row, _col]) => row === _row && col === _col)) {
      mines.push([row, col]);
      console.log(`Mine placed on [${row}, ${col}]`);
    } else {
      console.log(`Mine already on [${row}, ${col}], rerolling...`);
    }
  }

  let boardReveals: boolean[][] = Array(HEIGHT)
    .fill(undefined)
    .map(() => Array(WIDTH).fill(false));

  const getCellNumber = (row: number, col: number) =>
    mines.filter(
      ([_row, _col]) =>
        _row >= row - 1 && _row <= row + 1 && _col >= col - 1 && _col <= col + 1
    ).length;

  const hasMine = (row: number, col: number) =>
    mines.some(([_row, _col]) => _row === row && _col === col);

  const revealCell = (row: number, col: number) => {
    if (row < 0 || col < 0 || row >= HEIGHT || col >= WIDTH) return;
    if (boardReveals[row][col]) return;

    boardReveals = boardReveals.map((rowCells, _row) =>
      rowCells.map((revealedCell, _col) =>
        _row === row && _col === col ? true : revealedCell
      )
    );

    if (hasMine(row, col)) endGame = true;

    if (getCellNumber(row, col) === 0) {
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
</script>

<main class:endGame>
  {#each boardReveals as boardRow, row (row)}
    <div class="row">
      {#each boardRow as cellIsRevealed, col (col)}
        {#if cellIsRevealed}
          {#if hasMine(row, col)}
            <div class="square revealed">*</div>
          {:else}
            <div class="square revealed">{getCellNumber(row, col)}</div>
          {/if}
        {:else}
          <!-- svelte-ignore a11y-no-static-element-interactions -->
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          <div class="square" on:click={() => revealCell(row, col)} />
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
  }
  .square {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    height: 50px;
    width: 50px;
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
