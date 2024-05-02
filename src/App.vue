<script setup>
import intersection from './components/roadIntersection.vue'
import { ref } from 'vue'

const IntersectionType = ref('+')
const forceUpdate = ref(0)
const updateType = (event) => {
  IntersectionType.value = event.target.value
  forceUpdate.value = (forceUpdate.value + 1) % 2
}
</script>

<template>
  <select name="type" id="select" @input="updateType($event)">
    <option value="+">+</option>
    <option value="T">T</option>
  </select>
  <intersection
    class="intersection"
    :type="IntersectionType"
    priority-road="RB"
    :key="forceUpdate"
    cars="RLTB"
  ></intersection>
</template>

<style lang="scss" scoped>
.intersection {
  width: 800px;
  height: 700px;
  overflow: hidden;
}
@media (min-width: 1024px) {
}
</style>
