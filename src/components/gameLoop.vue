<script setup>
import intersection from './roadIntersection.vue'
import { ref } from 'vue'

const props = defineProps({
  playerCar: {
    type: String,
    required: true,
    validator(value) {
      return '1234'.includes(value)
    },
    default: '1'
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
  if (cars === 'B' || (_type === '+' && cars.length == 2))
    cars += structure.split('').filter((e) => e != 'B')[getRandInt(1)]

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

const calculatePriorityCars = () => {
  const interData = intersectionData.value
  let cars = []
  //if on priority road
  if (interData.priorityRoad.includes('B')) {
    //if stay on priority road or turn right
    if (
      interData.priorityRoad.includes('R') ||
      interData.priorityRoad.split('').includes(intersectionData.value.direction)
    )
      return []
    //if other car is on priority road
    else if (interData.cars.includes(interData.priorityRoad.split('').filter((e) => e != 'B'))[0]) {
      cars.push(interData.priorityRoad.split('').filter((e) => e != 'B')[0])
      console.log('OTHER CAR ON PRIORITY ROAD')
    }
  }
  return cars
}

let selectedCars = ref([])
const updateSelectedCars = (selectCarEmit) => {
  selectedCars.value = selectCarEmit.value
}

let _PREV
const refreshVal = ref(0)
const submit = () => {
  if (calculatePriorityCars().sort().join(',') === selectedCars.value.sort().join(','))
    console.log('ok')
  // else console.log('not ok')

  //generate new intersection
  selectedCars.value = []
  _PREV = intersectionData.value
  intersectionData.value = generateIntersection()
  refreshVal.value = (refreshVal.value + 1) % 2
}

const previous = () => {
  selectedCars.value = []
  intersectionData.value = _PREV
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
      :cars="intersectionData.cars"
      :selected="selectedCars.join('')"
      :player-car="props.playerCar"
      :direction="intersectionData.direction"
      @carSelect="updateSelectedCars"
      :key="refreshVal"
    ></intersection>
    <button type="button" @click="submit()">submit</button>
    <button type="previous" @click="previous()">go back</button>
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
