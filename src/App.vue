<script setup>
import { ref } from 'vue';

function generateSudoku(board = []) {
  if(board.length === 0) board = generateSolveableSudoku();

  let count = 0;

  while (true) {

    count++
    
    const emptyCells = shuffle(getEmptyCells(board));
    
    let uniqueSolution = true;

    for (let i = 0; i < emptyCells.length; i++) {

      const [row, col] = emptyCells[i];

      const value = board[row][col];
      board[row][col] = 0;

      if (!hasUniqueSolution(board)) {

        console.log(`Failed solve: ${count}`)
    
        board[row][col] = value;
        uniqueSolution = false;
        break;
      }

    }

    if (uniqueSolution) {
      const cells = [];

      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          const value = board[row][col];
          const readonly = value !== 0;
          const selected = false;

          cells.push({
            value,
            readonly,
            selected,
          });
        }
      }

      return ref(cells);
    }
  }
}

function generateSolveableSudoku() {
  const board = Array.from({ length: 9 }, () =>
    Array.from({ length: 9 }, () => 0)
  );
  backtrack(board, 0, 0);
  return board;
}

function shuffle(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
}

function getEmptyCells(board) {
  const cells = [];

  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      if (board[row][col] === 0) {
        cells.push([row, col]);
      }
    }
  }

  return cells;
}

function hasUniqueSolution(board) {
  return countSolutions(board) === 1;
}

function backtrack(board, row, col) {
  if (row === 9) {
    return true; // Reached the end of the board, so the puzzle is solveable
  }

  let nextRow = row;
  let nextCol = col + 1;

  if (nextCol === 9) {
    nextRow += 1;
    nextCol = 0;
  }

  const values = shuffle([1, 2, 3, 4, 5, 6, 7, 8, 9]);

  for (let i = 0; i < values.length; i++) {
    const value = values[i];

    if (isValidValue(board, row, col, value)) {
      board[row][col] = value;

      if (backtrack(board, nextRow, nextCol)) {
        return true; // Found a solution
      }

      board[row][col] = 0; // Backtrack
    }
  }

  return false; // No valid value found, so backtrack further
}

function isValidValue(board, row, col, value) {
  for (let i = 0; i < 9; i++) {
    if (board[row][i] === value || board[i][col] === value) {
      return false;
    }
  }

  const boxRow = Math.floor(row / 3) * 3;
  const boxCol = Math.floor(col / 3) * 3;

  for (let i = boxRow; i < boxRow + 3; i++) {
    for (let j = boxCol; j < boxCol + 3; j++) {
      if (board[i][j] === value) {
        return false;
      }
    }
  }

  return true;
}

function countSolutions(board) {
  let count = 0;

  backtrack(board, 0, 0);

  function backtrack(board, row, col) {
    if (row === 9) {
      count += 1; // Found a solution, so increment the count
      return;
    }

    let nextRow = row;
    let nextCol = col + 1;

    if (nextCol === 9) {
      nextRow += 1;
      nextCol = 0;
    }

    if (board[row][col] !== 0) {
      backtrack(board, nextRow, nextCol);
    } else {
      const values = shuffle([1, 2, 3, 4, 5, 6, 7, 8, 9]);

      for (let i = 0; i < values.length; i++) {
        const value = values[i];

        if (isValidValue(board, row, col, value)) {
          board[row][col] = value;

          backtrack(board, nextRow, nextCol);

          board[row][col] = 0; // Backtrack
        }
      }
    }
  }

  return count;
}

function selectCell(cell, slot) {

  if (selectedCell.value) {
    selectedCell.value.selected = false;
  }

  cell.selected = true;
  selectedCell.value = cell;

  console.log(selectedCell.value, slot)
}

function generatePuzzle(board, n) {
  // Remove n numbers from the board while ensuring that the resulting
  // puzzle remains solvable
  let count = 0;
  let loop = 0;
  while (count < n) {
    const row = Math.floor(Math.random() * 9);
    const col = Math.floor(Math.random() * 9);

    loop++

    if (board[row][col] === 0) {
      continue;
    }
    
    const savedValue = board[row][col];

    board[row][col] = 0;

    if (countSolutions(board) === 1) {
      count++;

    } else {

      board[row][col] = savedValue;
    }
  }
  return board;
}

function solveSudoku() {
  console.log('solve');
}

function handleKeydown(event) {
  console.log('xxx')
  console.log(event)
}

let board = generateSolveableSudoku()

generatePuzzle(board, 53)

let cells = generateSudoku(board);

let selectedCell = ref(null);
</script>

<template>
  <div class="flex flex-col items-center justify-center">
    <h1 class="text-3xl font-bold text-center mb-8 underline">Sudoku Game</h1>
    <div
      class="grid grid-cols-9 grid-rows-9 border-2 border-black w-96 h-96 mb-8"
    >
      <div
        v-for="(cell, index) in cells"
        :key="index"
        class="border border-black flex justify-center items-center font-bold text-2xl cursor-pointer"
        :class="{
          'border-r-2': (index + 1) % 3 === 0,
          'border-b-2':
            (index + 1 > 18 && index + 1 <= 27) ||
            (index + 1 > 45 && index + 1 <= 54),
          'border-l': (index + 1) % 3 === 0 && (index + 1) % 9 !== 0,
          'border-t': index >= 27 && index % 9 !== 0,
          'bg-gray-100': cell.readonly,
          'bg-yellow-200': cell.selected,
        }"
        @click="selectCell(cell, index + 1)"
        @keydown="handleKeydown()"
      >
        {{ cell.value || '' }}
      </div>
    </div>

    <button
      class="block mx-auto px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
      @click="solveSudoku()"
    >
      Solve
    </button>
  </div>
</template>
