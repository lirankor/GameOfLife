<template>
  <div class="game-controls">
    <button @click="play" :disabled="isRunning">Play</button>
    <button @click="next" :disabled="isRunning">Next</button>
    <button @click="stop" :disabled="!isRunning">Stop</button>

    <div class="slider-control">
      <label for="speed">Speed:</label>
      <input
        type="range"
        id="speed"
        min="50"
        max="1000"
        step="50"
        v-model="speedValue"
        :disabled="isRunning"
      />
      <span>{{ speedValue }}ms</span>
    </div>

    <div class="slider-control">
      <label for="gridSize">Grid Size:</label>
      <input
        type="range"
        id="gridSize"
        min="10"
        max="200"
        v-model="gridSizeValue"
        :disabled="isRunning"
      />
      <span>{{ gridSizeValue }}X{{ gridSizeValue }}</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

const props = defineProps<{
  isRunning: boolean
  interval: number
  gridSize: number
}>()

const emit = defineEmits(['play', 'next', 'stop', 'update:interval', 'update:gridSize'])

const speedValue = ref(props.interval)
const gridSizeValue = ref(props.gridSize)

watch(speedValue, (newValue) => {
  emit('update:interval', Number(newValue))
})

watch(gridSizeValue, (newValue) => {
  emit('update:gridSize', Number(newValue))
})

const play = () => {
  emit('play')
}

const next = () => {
  emit('next')
}

const stop = () => {
  emit('stop')
}
</script>

<style scoped>
.game-controls {
  margin: 20px 0;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  background-color: #f0f0f0;
  width: 770px;
  padding: 20px;
  border-radius: 10px;
  margin: 20px auto;
}

button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  background-color: #4caf50;
  color: white;
  cursor: pointer;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.slider-control {
  display: flex;
  align-items: center;
  gap: 8px;
  input[type='range'] {
    accent-color: #4caf50;
  }
}

input[type='range'] {
  width: 150px;
}

input[type='number'] {
  width: 60px;
}
</style>
