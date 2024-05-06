<script setup>
import intersection from './roadIntersection.vue'
import { ref } from 'vue'

const props = defineProps({
  playerCar: {
    type: String,
    required: true,
    validator(value) {
      return '1234'.includes(value)
    }
  }
})

const getRandInt = (max) => Math.floor(Math.random() * (max + 1))

const generateStructure = (intersectionType) => {
  if (intersectionType == '+') {
    return 'RTLB'
  }
  return ['RLB', 'RTB', 'LTB'][getRandInt(2)]
}

const generateIntersection = () => {
  const _type = getRandInt(2) == 0 ? '+' : 'T'
  const structure = generateStructure(_type)

  //priority road
  let r1 = ''
  let r2 = ''
  do {
    r1 = structure[getRandInt(structure.length - 1)]
    r2 = structure[getRandInt(structure.length - 1)]
  } while (r1 === r2)
  const priorityRd = r1 + r2

  //cars
  let cars = 'B'
  structure
    .split('')
    .filter((e) => e != 'B')
    .forEach((pos) => {
      if (getRandInt(2) > 0) cars += pos
    })
  if (cars === 'B') cars += structure.split('').filter((e) => e != 'B')[getRandInt(1)]

  //stop signs
  let stops = ''
  structure
    .split('')
    .filter((e) => !priorityRd.includes(e))
    .forEach((pos) => {
      if (getRandInt(3) == 0) stops += pos
    })

  //direction
  const direction = structure.split('').filter((e) => e != 'B')[getRandInt(structure.length - 2)]
  console.log(direction)
  return {
    type: _type,
    structure: structure,
    priorityRoad: priorityRd,
    cars: cars,
    stopSigns: stops,
    direction: direction
  }
}
const intersectionData = ref(generateIntersection())

const refreshVal = ref(0)
const refresh = () => {
  intersectionData.value = generateIntersection()
  refreshVal.value = (refreshVal.value + 1) % 2
}
</script>

<template>
  <div class="wrapper">
    <intersection
      class="intersectionDisplay"
      :type="intersectionData.type"
      :structure="intersectionData.structure"
      :priority-road="intersectionData.priorityRoad"
      :stop-signs="intersectionData.stopSigns"
      :key="refreshVal"
      :cars="intersectionData.cars"
      :player-car="props.playerCar"
      :direction="intersectionData.direction"
    ></intersection>
    <button type="button" @click="refresh()">generate</button>
  </div>
</template>

<style lang="scss" scoped>
.intersectionDisplay {
  width: 800px;
  height: 700px;
  overflow: hidden;
}
@media (min-width: 1024px) {
}
</style>
