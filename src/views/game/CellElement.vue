<template>
  <div class="cell" :class="{ alive: props.cell.isAlive }" @click="handleClick"></div>
</template>

<script setup lang="ts">
import type { Cell } from './types'

const props = defineProps<{
  cell: Cell
  onCellClick: (cell: Cell) => void
}>()

const handleClick = () => {
  props.onCellClick(props.cell)
}
</script>
