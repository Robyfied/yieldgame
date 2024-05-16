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
  <startMenu v-if="gameState === 'startMenu'" @gameStart="gameState = 'chooseCar'"></startMenu>
  <chooseCar v-else-if="gameState === 'chooseCar'" @carSelection="handleCarSelection"></chooseCar>
  <gameLoop v-else-if="gameState === 'gameLoop'" :playerCar="chosenCar"></gameLoop>
</template>

<style lang="scss" scoped>
* {
  user-select: none;
}
</style>
