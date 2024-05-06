<script setup></script>

<template>
  <div class="wrapper">
    <button type="button" class="startButton" @click="$emit('gameStart')">Start Game</button>
  </div>
</template>

<style lang="scss" scoped>
@import '../assets/base.scss';

.startButton {
  font-family: SilkScreen, Verdana, Geneva, Tahoma, sans-serif;
  padding: 2rem 5rem;
  font-size: 2rem;
}
</style>
