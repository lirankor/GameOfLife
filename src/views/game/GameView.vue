<template>
  <div>
    <h1>Game of Life</h1>

    <GameControls
      :is-running="isRunning"
      :interval="interval"
      :grid-size="gridSize"
      @play="startGame"
      @next="calculateNextGrid"
      @stop="stopGame"
      @update:interval="interval = $event"
      @update:grid-size="updateGridSize"
    />

    <div class="grid-container" :style="gridContainerStyle">
      <div class="grid" :style="gridStyle">
        <CellElement
          v-for="[key, cell] in grid"
          :key="key"
          :cell="cell"
          :onCellClick="handleCellClick"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, reactive, computed } from 'vue'
import type { Cell } from './types'
import CellElement from './CellElement.vue'
import GameControls from './ActionBar.vue'

const gridSize = ref<number>(67)
const interval = ref<number>(400)
const iteration = ref<number>(0)
const isRunning = ref<boolean>(iteration.value > 0)
const gameInterval = ref<number | null>(null)

const grid = reactive(new Map<string, Cell>())
const nextGrid = reactive(new Map<string, Cell>())

const computedGridSize = computed(() => gridSize.value + 40)

// init grid (all cells are dead)
// TODO: add templates
const initGrid = () => {
  for (let i = 0; i < computedGridSize.value; i++) {
    for (let j = 0; j < computedGridSize.value; j++) {
      const cell = <Cell>{
        posistion: {
          x: i,
          y: j,
        },
        isAlive: false,
      }
      grid.set(`${i},${j}`, cell)
    }
  }
  console.log(computedGridSize)
}

// handle cell click
const handleCellClick = (cell: Cell) => {
  if (isRunning.value) return
  cell.isAlive = !cell.isAlive
  grid.set(`${cell.posistion.x},${cell.posistion.y}`, cell)
}

const getNeighbors = (cell: Cell) => {
  const neighbors = <Cell[]>[
    grid.get(`${cell.posistion.x - 1},${cell.posistion.y - 1}`),
    grid.get(`${cell.posistion.x - 1},${cell.posistion.y}`),
    grid.get(`${cell.posistion.x - 1},${cell.posistion.y + 1}`),
    grid.get(`${cell.posistion.x},${cell.posistion.y - 1}`),
    grid.get(`${cell.posistion.x},${cell.posistion.y + 1}`),
    grid.get(`${cell.posistion.x + 1},${cell.posistion.y - 1}`),
    grid.get(`${cell.posistion.x + 1},${cell.posistion.y}`),
    grid.get(`${cell.posistion.x + 1},${cell.posistion.y + 1}`),
  ]
  return neighbors
}

const calculateNextGrid = () => {
  // get all alive cells
  const cellsToCheck: Cell[] = Array.from(grid.values()).filter((cell) => cell.isAlive)
  grid.forEach((cell, key) => nextGrid.set(key, cell))

  const aliveCells = [...cellsToCheck]

  // get all neighbors of alive cells
  for (const cell of aliveCells) {
    const neighbors = getNeighbors(cell)
    cellsToCheck.push(...neighbors)
  }

  // calculate next grid
  for (const cell of cellsToCheck) {
    const neighbors = getNeighbors(cell)
    const aliveNeighbors = neighbors.filter((neighbor) => neighbor?.isAlive).length
    //eliminate cells that are outside the grid
    if (
      cell.posistion.x < 0 ||
      cell.posistion.x >= computedGridSize.value ||
      cell.posistion.y < 0 ||
      cell.posistion.y >= computedGridSize.value
    ) {
      nextGrid.delete(`${cell.posistion.x},${cell.posistion.y}`)
      continue
    }
    if (cell.isAlive) {
      if (aliveNeighbors < 2 || aliveNeighbors > 3) {
        nextGrid.set(`${cell.posistion.x},${cell.posistion.y}`, { ...cell, isAlive: false })
      }
    }
    if (!cell.isAlive) {
      if (aliveNeighbors === 3) {
        nextGrid.set(`${cell.posistion.x},${cell.posistion.y}`, { ...cell, isAlive: true })
      }
    }
  }
  iteration.value++
  grid.clear()
  nextGrid.forEach((cell, key) => grid.set(key, cell))
  nextGrid.clear()
}

const startGame = () => {
  stopGame()
  isRunning.value = true
  gameInterval.value = setInterval(() => {
    calculateNextGrid()
  }, interval.value)
}

const stopGame = () => {
  if (gameInterval.value) {
    clearInterval(gameInterval.value)
    gameInterval.value = null
    isRunning.value = false
    iteration.value = 0
    grid.clear()
    initGrid()
  }
}

const updateGridSize = (size: number) => {
  stopGame()
  gridSize.value = size
  grid.clear()
  nextGrid.clear()
  initGrid()
}

const gridStyle = computed(() => ({
  width: `${computedGridSize.value * 12}px`,
  gridTemplateColumns: `repeat(${computedGridSize.value}, 1fr)`,
}))

const gridContainerStyle = computed(() => ({
  width: `${gridSize.value * 12}px`,
  height: `${gridSize.value * 12}px`,
}))

// stop game
onMounted(() => {
  initGrid()
})
</script>
