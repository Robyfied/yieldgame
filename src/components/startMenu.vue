<script setup>
import intersection from './roadIntersection.vue'
</script>

<template>
  <div class="wrapper">
    <div class="content">
      <button type="button" class="startButton" @click="$emit('gameStart')">Incepe Jocul</button>
    </div>
    <div class="bgContainer">
      <intersection
        class="intersectionBG"
        type="+"
        cars=""
        priority-road="RL"
        stop-signs="B"
      ></intersection>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@import '../assets/base.scss';
@keyframes intersectionSpin {
  from {
    transform: rotateX(65deg) rotateZ(0deg);
  }
  to {
    transform: rotateX(65deg) rotateZ(360deg);
  }
}

.startButton {
  position: relative;
  font-family: SilkScreen, Verdana, Geneva, Tahoma, sans-serif;
  padding: 2rem 5rem;
  font-size: 2rem;
  background: rgba(255, 249, 246, 0.5);
  border: 2px solid black;
  border-radius: 1.5rem;
  transition: all 1s cubic-bezier(0.075, 0.82, 0.165, 1);
  filter: drop-shadow(0.2rem 0.2rem 0.2rem rgba(0, 0, 0, 0.5));

  &:hover {
    transform: scale(1.08);
    background: rgba(255, 249, 246, 1);
    border: 5px solid #ffc8b1;
    filter: drop-shadow(0rem 0rem 0.5rem rgba(255, 255, 255, 0.5));
  }
  z-index: 100;
}

.bgContainer {
  position: fixed;
  left: 0;
  top: 0;
  overflow: hidden;
  height: 100vh;
  width: 100vw;
}

.intersectionBG {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translateX(-50%) translateY(-50%);
  transform-origin: center center;
  filter: opacity(0.5) blur(0.3em);
  width: 90rem;
  height: 90rem;
  overflow: hidden;
  perspective: 75rem;

  &::v-deep {
    .intersection {
      animation: intersectionSpin 30s linear infinite;
    }

    .signWrapper::before {
      bottom: -18%;
    }
  }
}
</style>
