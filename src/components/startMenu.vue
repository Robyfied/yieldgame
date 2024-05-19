<script setup>
import intersection from './roadIntersection.vue'
</script>

<template>
  <div class="wrapper">
    <div class="contentWrapper">
      <div class="titleWrapper">
        <img src="/title.png" alt="title" class="title" />
      </div>
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

* {
  user-select: none;
}

.contentWrapper {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4rem;
  background-color: rgba(255, 255, 255, 0.2);
  padding: 3rem 8rem;
  border-radius: 2rem;
  z-index: 5;
}

.title {
  width: 40rem;
}

.startButton {
  position: relative;
  font-family: SilkScreen, Verdana, Geneva, Tahoma, sans-serif;
  padding: 4% 8%;
  font-size: 2.5rem;
  background: none;
  border: none;
  outline: 0.3rem solid black;
  color: black;
  transition: all 1s cubic-bezier(0.075, 0.82, 0.165, 1);
  filter: drop-shadow(0.2rem 0.2rem 0.2rem rgba(0, 0, 0, 0.5));

  &:hover {
    transform: scale(1.08);
    background: rgba(255, 249, 246, 1);
    outline: 5px solid #ffc8b1;
    color: #ffc8b1;
    filter: drop-shadow(0rem 0rem 0.5rem rgba(255, 255, 255, 0.5));
  }
}

.bgContainer {
  position: fixed;
  left: 0;
  top: 0;
  overflow: hidden;
  height: 100vh;
  filter: opacity(0.4);
  width: 100vw;
}

.intersectionBG {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translateX(-50%) translateY(-50%);
  transform-origin: center center;
  width: 100%;
  height: 100%;
  overflow: hidden;
  perspective: 150rem;

  &:deep {
    .intersection {
      animation: intersectionSpin 30s linear infinite;
    }

    .signWrapper::before {
      bottom: -18%;
    }
  }
}

@media (max-width: 1024px) {
  .contentWrapper {
    background: none;
  }

  .title {
    width: 60vw;
  }

  .startButton {
    width: 50vw;
    font-size: 4.5vw;
    padding: 5%;
  }

  .intersectionBG {
    opacity: 0.5;
    width: 200%;
  }
}
</style>
