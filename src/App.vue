<script setup>
import { ref } from 'vue';

const gridItems = ref(
  [
    [
      { label: '1 x 1', color: false },
      { label: '1 x 2', color: false },
      { label: '1 x 3', color: false },
      { label: '1 x 4', color: false },
      { label: '1 x 5', color: false },
      { label: '1 x 6', color: false },
      { label: '1 x 7', color: false },
      { label: '1 x 8', color: false },
      { label: '1 x 9', color: false },
    ],
    [
      { label: '2 x 1', color: false },
      { label: '2 x 2', color: false },
      { label: '2 x 3', color: false },
      { label: '2 x 4', color: false },
      { label: '2 x 5', color: false },
      { label: '2 x 6', color: false },
      { label: '2 x 7', color: false },
      { label: '2 x 8', color: false },
      { label: '2 x 9', color: false },
    ],
    [
      { label: '3 x 1', color: false },
      { label: '3 x 2', color: false },
      { label: '3 x 3', color: false },
      { label: '3 x 4', color: false },
      { label: '3 x 5', color: false },
      { label: '3 x 6', color: false },
      { label: '3 x 7', color: false },
      { label: '3 x 8', color: false },
      { label: '3 x 9', color: false },
    ],
    [
      { label: '4 x 1', color: false },
      { label: '4 x 2', color: false },
      { label: '4 x 3', color: false },
      { label: '4 x 4', color: false },
      { label: '4 x 5', color: false },
      { label: '4 x 6', color: false },
      { label: '4 x 7', color: false },
      { label: '4 x 8', color: false },
      { label: '4 x 9', color: false },
    ],
    [
      { label: '5 x 1', color: false },
      { label: '5 x 2', color: false },
      { label: '5 x 3', color: false },
      { label: '5 x 4', color: false },
      { label: '5 x 5', color: false },
      { label: '5 x 6', color: false },
      { label: '5 x 7', color: false },
      { label: '5 x 8', color: false },
      { label: '5 x 9', color: false },
    ],
    [
      { label: '6 x 1', color: false },
      { label: '6 x 2', color: false },
      { label: '6 x 3', color: false },
      { label: '6 x 4', color: false },
      { label: '6 x 5', color: false },
      { label: '6 x 6', color: false },
      { label: '6 x 7', color: false },
      { label: '6 x 8', color: false },
      { label: '6 x 9', color: false },
    ],
    [
      { label: '7 x 1', color: false },
      { label: '7 x 2', color: false },
      { label: '7 x 3', color: false },
      { label: '7 x 4', color: false },
      { label: '7 x 5', color: false },
      { label: '7 x 6', color: false },
      { label: '7 x 7', color: false },
      { label: '7 x 8', color: false },
      { label: '7 x 9', color: false },
    ],
    [
      { label: '8 x 1', color: false },
      { label: '8 x 2', color: false },
      { label: '8 x 3', color: false },
      { label: '8 x 4', color: false },
      { label: '8 x 5', color: false },
      { label: '8 x 6', color: false },
      { label: '8 x 7', color: false },
      { label: '8 x 8', color: false },
      { label: '8 x 9', color: false },
    ],
    [
      { label: '9 x 1', color: false },
      { label: '9 x 2', color: false },
      { label: '9 x 3', color: false },
      { label: '9 x 4', color: false },
      { label: '9 x 5', color: false },
      { label: '9 x 6', color: false },
      { label: '9 x 7', color: false },
      { label: '9 x 8', color: false },
      { label: '9 x 9', color: false },
    ],
  ]
);

const handleClick = (row, col) => {
  const maxRow = gridItems.value.length - 1;
  const maxCol = gridItems.value[0].length - 1;
  const maxDistance = Math.min(row, maxRow - row, col, maxCol - col);

  for (let i = 0; i <= maxDistance; i++) {
    const leftCol = col - i;
    const rightCol = col + i;
    const topRow = row - i;
    const bottomRow = row + i;

    if (leftCol >= 0) {
      gridItems.value[row][leftCol].color = true;
    }
    if (rightCol <= maxCol && i !== 0) {
      gridItems.value[row][rightCol].color = true;
    }
    if (topRow >= 0) {
      gridItems.value[topRow][col].color = true;
    }
    if (bottomRow <= maxRow && i !== 0) {
      gridItems.value[bottomRow][col].color = true;
    }
  }
};

// const handleClick = (row, col) => {

//   console.log(row, col)

//   // Set the clicked item's color to pink
//   gridItems.value[row].color = !gridItems.value[row].color;

//   // Change color of neighboring items

//   for (let i = 0; i < 9; i++) {
//     setTimeout(() => {
//       if (row > i) {
//         gridItems.value[row - (i + 1)].color = !gridItems.value[row - (i + 1)].color;
//       }
//       if (row < gridItems.value.length - (i + 1)) {
//         gridItems.value[row + (i + 1)].color = !gridItems.value[row + (i + 1)].color;
//       }
//     }, 100 * (i + 1));
//   }

// };
</script>

<template>
  <div class="grid grid-cols-9 gap-4 bg-white">
  <div v-for="(cols, col_index) in gridItems">
    <div
      v-for="(row, row_index) in cols"
      :key="col_index"
      @click="handleClick(row_index, col_index)"
      :class="{ 
        'bg-pink-500': row.color,
        'bg-white': !row.color 
        }"
      class="rounded-md h-12 flex items-center justify-center cursor-pointer"
    >
      {{ row.label }}
    </div>
  </div>
  </div>
</template>
