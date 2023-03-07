<script setup>
import { onMounted, ref } from 'vue';

function generateSudoku(board = []) {
  if (board.length === 0) board = generateSolveableSudoku();

  let count = 0;

  while (true) {
    count++;

    const emptyCells = shuffle(getEmptyCells(board));

    let uniqueSolution = true;

    for (let i = 0; i < emptyCells.length; i++) {
      const [row, col] = emptyCells[i];

      const value = board[row][col];
      board[row][col] = 0;

      if (countSolutions(board) !== 1) {
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
          let value_status = '';
          if (value !== 0) value_status = 'default'
          const readonly = value !== 0;
          const selected = false;

          cells.push({
            row,
            col,
            value,
            readonly,
            value_status,
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
}

function generatePuzzle(generated_board, n) {
  let puzzle = false;

  while (!puzzle) {
    try {
      // Start the timer
      const start = performance.now();

      let count = 0;
      let loop = 0;
      while (count < n) {
        const row = Math.floor(Math.random() * 9);
        const col = Math.floor(Math.random() * 9);

        loop++;

        if (generated_board[row][col] === 0) {
          continue;
        }

        const savedValue = generated_board[row][col];

        generated_board[row][col] = 0;

        if (countSolutions(generated_board) === 1) {
          count++;

          if (count >= n) puzzle = true;
        } else {
          generated_board[row][col] = savedValue;
        }

        // Check if the time limit has been exceeded
        const elapsed = performance.now() - start;
        if (elapsed > 5000) {
          console.log(`time limit exceeded : ${elapsed}`);
          throw new Error('Generation time limit exceeded');
        }
      }
    } catch (error) {
      if (error.message === 'Generation time limit exceeded') {
        // Retry the generation process
        generated_board = generateSolveableSudoku();
        continue;
      } else {
        // Handle other errors
        throw error;
      }
    }
  }

  return generated_board;
}

function checkValidInput(key) {
  // pull valid board from solve function
  let valid_board = solveSudoku(board)

  if (valid_board[selectedCell.value.row][selectedCell.value.col] != key) {
    selectedCell.value.value_status = 'wrong'
    // selectedCell.value.value = 0
  } else {
    selectedCell.value.value_status = 'currect'
  }
  // console.log(valid_board, key, selectedCell.value)
}

function handleKeydown(event) {
  // console.log(event.key)
  if (selectedCell.value && (selectedCell.value.value == 0 || selectedCell.value.value_status == 'wrong')) {
    if (['1', '2', '3', '4', '5', '6', '7', '8', '9'].includes(event.key)) {
      selectedCell.value.value = parseInt(event.key);
      checkValidInput(event.key)
    }
    if (event.key == 'Backspace') { 
      selectedCell.value.value = 0
      selectedCell.value.value_status = ''
    }

    updateNumberTotal();
  }
}

function updateNumberTotal() {
  const obj = {};
  for (let i = 1; i <= 9; i++) {
    obj[`num_${i}`] = cells.value
      .map((item) => item.value)
      .filter((num) => num === i).length;
  }

  numberTotal.value = obj;
}

///
function solveSudoku(board) {
  const size = 9;

  function getRow(row) {
    return board[row];
  }

  function getColumn(col) {
    return Array.from({ length: size }, (_, i) => board[i][col]);
  }

  function getBox(row, col) {
    const boxSize = 3;
    const startRow = Math.floor(row / boxSize) * boxSize;
    const startCol = Math.floor(col / boxSize) * boxSize;
    const box = [];

    for (let i = 0; i < boxSize; i++) {
      for (let j = 0; j < boxSize; j++) {
        box.push(board[startRow + i][startCol + j]);
      }
    }

    return box;
  }

  function getPossibleValues(row, col) {
    const used = new Set([
      ...getRow(row),
      ...getColumn(col),
      ...getBox(row, col),
    ]);
    const possible = new Set([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    for (const val of used) {
      possible.delete(val);
    }

    return possible;
  }

  function solve() {
    for (let row = 0; row < size; row++) {
      for (let col = 0; col < size; col++) {
        if (board[row][col] === 0) {
          const possibleValues = Array.from(getPossibleValues(row, col));

          for (const value of possibleValues) {
            board[row][col] = value;

            if (solve()) {
              return true;
            }

            board[row][col] = 0;
          }

          return false;
        }
      }
    }

    return true;
  }

  solve();
  return board;
}
///

function writeAnswers(board) {
  // loop all the original board then place the number that not 0
  let answers = solveSudoku(board);

  animateSolution(answers);
}

function animateSolution(answers) {
  const size = 9;
  const delay = 100;

  let index = 0;

  // get all value = 0 in cells and collect index, row and col position in Arr
  let zeroCells = cells.value
    .map((item, index) => ({ index: index, value: item }))
    .filter((item) => item.value.value == 0);

  // Shuffle the copy of the array
  for (let i = zeroCells.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [zeroCells[i], zeroCells[j]] = [zeroCells[j], zeroCells[i]];
  }

  function fillCell() {
    // loop shuffled arr then fill value
    if (index >= zeroCells.length) {
      clearInterval(interval);
      return;
    }

    cells.value[zeroCells[index].index].value = answers[zeroCells[index].value.row][zeroCells[index].value.col];
    cells.value[zeroCells[index].index].value_status = 'currect'

      updateNumberTotal();

    index++;
  }

  const interval = setInterval(fillCell, delay);
}

function newGame() {
  location.reload()
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown);
});

let selectedCell = ref({});

console.time('generate-puzzle');
let board = generateSolveableSudoku();
generatePuzzle(board, 40);
let cells = generateSudoku(board);
console.timeEnd('generate-puzzle');

let numberTotal = ref({});

updateNumberTotal();
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
          'bg-orange-100':
            selectedCell &&
            (selectedCell.col === cell.col || selectedCell.row === cell.row),
          'text-orange-400': selectedCell && selectedCell.value === cell.value,
          'bg-green-200': !cell.readonly && cell.value != 0 && cell.value_status == 'currect',
          'bg-red-400 text-red-900 font-bold': cell.value_status == 'wrong',
          'bg-gray-300': cell.readonly,
          'bg-yellow-200': cell.selected,
        }"
        @click="selectCell(cell, index + 1)"
      >
        <span :class="{'invisible': !cell.value}">{{ cell.value }}</span>
      </div>
    </div>

    <div
      class="w-full mb-10 bg-blue-100 border-2 border-black py-4 px-4 rounded-xl flex justify-around items-center"
    >
      <div v-for="i in 9">
        <div
          class="bg-gray-400 text-white font-bold py-2 px-2 rounded-md"
          :class="{
            'bg-orange-500': selectedCell && selectedCell.value === i,
            'bg-green-600': numberTotal[`num_${i}`] == 9,
          }"
        >
          {{ i }} : {{ numberTotal[`num_${i}`] }}
        </div>
      </div>
    </div>

<div class="flex space-x-2">
    <button
      class="block mx-auto px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
      @click="writeAnswers(board)"
    >
      Solve
    </button>
    <button
      class="block mx-auto px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2"
      @click="newGame()"
    >
      New Game
    </button>
</div>
  </div>
</template>
