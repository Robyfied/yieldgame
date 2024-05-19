<script setup>
import gameLoop from './components/gameLoop.vue'
import startMenu from './components/startMenu.vue'
import chooseCar from './components/chooseCar.vue'
import { ref } from 'vue'

const gameState = ref('startMenu')

const chosenCar = ref('')
const handleCarSelection = (e) => {
  chosenCar.value = e
  gameState.value = 'gameLoop'
}
</script>

<template>
  <startMenu
    v-if="gameState === 'startMenu'"
    class="fadeInAnimation"
    @gameStart="gameState = 'chooseCar'"
  ></startMenu>
  <chooseCar
    v-else-if="gameState === 'chooseCar'"
    class="fadeInAnimation"
    @carSelection="handleCarSelection"
  ></chooseCar>
  <gameLoop
    v-else-if="gameState === 'gameLoop'"
    class="fadeInAnimation"
    :playerCar="chosenCar"
    @returnToStart="gameState = 'startMenu'"
  ></gameLoop>
  <div class="version">
    <span>Version ALPHA 1.0</span>
  </div>
</template>

<style lang="scss" scoped>
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.fadeInAnimation {
  animation: fadeIn 0.8s;
}

.version {
  position: absolute;
  right: 0;
  bottom: 0;
}
</style>
