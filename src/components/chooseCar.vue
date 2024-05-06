<script setup>
import { ref } from 'vue'
const carTextures = ref([
  '/car_textures/1/car1T.png',
  '/car_textures/2/car2T.png',
  '/car_textures/3/car3T.png',
  '/car_textures/4/car4T.png'
])

let intervalID = ref()
const animateRotation = (carNo) => {
  const carPositions = ['R', 'B', 'L', 'T']
  let ix = 0
  carTextures.value[carNo - 1] = carTextures.value[carNo - 1].replace('T', 'R')
  intervalID.value = setInterval(() => {
    carTextures.value[carNo - 1] = carTextures.value[carNo - 1].replace(
      carPositions[ix],
      carPositions[ix == 3 ? 0 : ix + 1]
    )
    if (ix == 3) ix = 0
    else ix++
  }, 700)
}
const stopRotation = (carNo) => {
  clearInterval(intervalID.value)
  carTextures.value[carNo - 1] = carTextures.value[carNo - 1].replace(/R|L|B/g, 'T')
}
</script>

<template>
  <div class="wrapper">
    <h1 class="title">Choose your car</h1>
    <ul class="carsList">
      <li
        class="carContainer"
        id="car1"
        @click="$emit('carSelection', '1')"
        @mouseenter="animateRotation(1)"
        @mouseleave="stopRotation(1)"
      >
        <img class="carTex" :src="carTextures[0]" alt="car1" draggable="false" />
      </li>
      <li
        class="carContainer"
        id="car2"
        @click="$emit('carSelection', '2')"
        @mouseenter="animateRotation(2)"
        @mouseleave="stopRotation(2)"
      >
        <img class="carTex" :src="carTextures[1]" alt="car2" draggable="false" />
      </li>
      <li
        class="carContainer"
        id="car3"
        @click="$emit('carSelection', '3')"
        @mouseenter="animateRotation(3)"
        @mouseleave="stopRotation(3)"
      >
        <img class="carTex" :src="carTextures[2]" alt="car3" draggable="false" />
      </li>
      <li
        class="carContainer"
        id="car4"
        @click="$emit('carSelection', '4')"
        @mouseenter="animateRotation(4)"
        @mouseleave="stopRotation(4)"
      >
        <img class="carTex" :src="carTextures[3]" alt="car4" draggable="false" />
      </li>
    </ul>
  </div>
</template>

<style lang="scss" scoped>
@import url('../assets/base.scss');

img {
  image-rendering: pixelated;
}

.title {
  position: relative;
  text-align: center;
  font-size: 3rem;
  font-family: SilkScreen, Verdana, Geneva, Tahoma, sans-serif;
  margin-bottom: 4rem;
  user-select: none;

  &::after {
    content: '';
    position: absolute;
    height: 0.5rem;
    width: 3rem;
    background-color: rgb(254, 55, 55);
    border-radius: 1rem;
    bottom: -1rem;
    left: 50%;
    transform: translateX(-50%);
  }
}
.carsList {
  list-style: none;
  display: flex;
  justify-content: center;
  flex-direction: row;
  flex-wrap: wrap;
  gap: 1rem;
}
.carContainer {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 9rem;
  width: 15rem;
  background: #fff9f6;
  border-radius: 1rem;
  border: 3px solid #ffc8b1;
  transition: all 1s cubic-bezier(0.075, 0.82, 0.165, 1);

  &:hover {
    transform: scale(1.08);
  }
}
.carTex {
  height: 4rem;
  object-fit: fill;
}
</style>
