<script setup>
import gameLoop from './components/gameLoop.vue'
import startMenu from './components/startMenu.vue'
import chooseCar from './components/chooseCar.vue'
import { ref } from 'vue'

const gameState = ref('startMenu')
const chosenCar = ref('1')
const handleCarSelection = (e) => {
  chosenCar.value = e
  gameState.value = 'gameLoop'
}
</script>

<template>
  <Transition>
    <startMenu v-if="gameState === 'startMenu'" @gameStart="gameState = 'chooseCar'"></startMenu>
    <chooseCar v-else-if="gameState === 'chooseCar'" @carSelection="handleCarSelection"></chooseCar>
    <gameLoop v-else-if="gameState === 'gameLoop'" :playerCar="chosenCar"></gameLoop>
  </Transition>
</template>

<style lang="scss" scoped>
* {
  user-select: none;
}

.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease-out;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}
</style>
