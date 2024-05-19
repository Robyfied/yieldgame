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
  const _type = getRandInt(1) == 0 ? '+' : 'T'
  const structure = generateStructure(_type)

  //priority road
  let r1 = ''
  let r2 = ''
  do {
    r1 = structure[getRandInt(structure.length - 1)]
    r2 = structure[getRandInt(structure.length - 1)]
    // 66% chance to reroll if player is on priority road
    if (r1 === 'B' && getRandInt(3) !== 0) {
      r1 = structure[getRandInt(structure.length - 1)]
    } else if (r2 === 'B' && getRandInt(3) !== 0) {
      r2 = structure[getRandInt(structure.length - 1)]
    }
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
  //if on priority road ↓
  if (interData.priorityRoad.includes('B')) {
    //if stay on priority road or turn right ↓
    if (interData.priorityRoad.split('').includes(intersectionData.value.direction)) return []
    //if leaving priority road and other car intersects ↓
    else if (interData.priorityRoad.includes('R') && interData.cars.includes('R'))
      cars.push(interData.priorityRoad.split('').filter((e) => e != 'B')[0])
    else if (
      interData.priorityRoad.includes('T') &&
      interData.cars.includes('T') &&
      interData.direction == 'L'
    )
      cars.push('T')
    else return []
  }
  // if not on priority road↓
  else {
    interData.priorityRoad.split('').forEach((e) => {
      if (interData.cars.includes(e)) cars.push(e)
    })

    // particular cases
    if (
      interData.direction === 'L' &&
      !interData.priorityRoad.includes('T') &&
      interData.cars.includes('T')
    )
      cars.push('T')
    if (
      !interData.priorityRoad.includes('R') &&
      interData.cars.includes('R') &&
      !cars.includes('R')
    )
      cars.push('R')
  }
  if (interData.direction === 'R') cars = cars.filter((e) => e != 'R') // really debatable honestly
  if (
    interData.type === 'T' &&
    ['TL', 'LT'].includes(interData.priorityRoad) &&
    interData.cars.includes('T')
  )
    cars = cars.filter((e) => e != 'T')
  return cars
}

let selectedCars = ref([])
const updateSelectedCars = (selectCarEmit) => {
  selectedCars.value = selectCarEmit.value
}

const getCorrectResult = () => {
  const priorityCars = calculatePriorityCars()
  if (priorityCars.length === 3) return 'toate cele 3 masini'

  const carPositionToName = {
    T: 'fata',
    R: 'dreapta',
    L: 'stanga'
  }

  let printVal = ''
  calculatePriorityCars().forEach((e) => {
    printVal += `masina din ${carPositionToName[e]} si `
  })
  return printVal === '' ? 'nicio masina' : printVal.substring(0, printVal.length - 3)
}

let _PREV
const correctResult = ref()
const result = ref('')
const score = ref(0)
const scoreMax = ref(0)
const refreshVal = ref(0)
const submit = () => {
  if (!infoDispaly.value) {
    if (calculatePriorityCars().sort().join(',') === selectedCars.value.sort().join(',')) {
      result.value = 'corect'
      score.value++
      if (score.value > scoreMax.value) scoreMax.value = score.value
    } else {
      result.value = 'incorect'
      correctResult.value = getCorrectResult()
      score.value = 0
    }

    //generate new intersection
    selectedCars.value = []
    _PREV = intersectionData.value
    intersectionData.value = generateIntersection()
    refreshVal.value = (refreshVal.value + 1) % 2
  }
}

const previous = () => {
  selectedCars.value = []
  intersectionData.value = _PREV
}

const infoDispaly = ref(true)
const toggleInfo = () => {
  infoDispaly.value = !infoDispaly.value
}
</script>

<template>
  <div class="wrapper">
    <div class="informationScreen" v-if="infoDispaly">
      <h1>Cum se joaca</h1>
      <img src="/carselect.gif" alt="car selection gif" id="carSelectGif" />
      <p>
        Pentru a castiga puncte, selectati <span>toate</span> masinile carora trebuie sa le cedati
        trecerea (din perspectiva masinii alese) si apasati butonul "verifica" pentru confirmare
      </p>
      <br />
      <p>
        <span style="font-size: 2rem">!</span> Puteti ceda trecerea
        <span>doar masinilor cu care va puteti intersecta</span>.
      </p>
      <br /><br />
      <button type="button" @click="toggleInfo()">Start</button>
    </div>
    <div class="gameContainer" :class="{ inactive: infoDispaly }">
      <h2>scor: {{ score }}</h2>
      <h4>maxim: {{ scoreMax }}</h4>
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
      <div class="footer">
        <div class="result">
          <span
            :class="{ correctResult: result === 'corect', incorrectResult: result === 'incorect' }"
            >{{ result }}</span
          >
          <div v-if="result === 'incorect'">
            <p>Raspunsul corect era: {{ correctResult }}</p>
            <button type="button" @click="previous()" id="retryBtn">incearca din nou</button>
          </div>
        </div>

        <div class="buttons">
          <button type="button" @click="$emit('returnToStart')" id="returnBtn">
            intoarce-te la meniu
          </button>
          <button type="button" @click="submit()" id="submitBtn">verifica</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@import '../assets/base.scss';

button {
  cursor: pointer;
}

.gameContainer {
  position: relative;
  display: flex;
  flex-direction: column;
}

.informationScreen {
  position: absolute;
  max-width: 60%;
  left: 50%;
  top: 50%;
  transform: translateX(-50%) translateY(-50%);
  z-index: 999999;
  background-color: white;
  border: 0.3rem solid black;
  padding: 3rem 5rem;

  h1 {
    margin-bottom: 3rem;
    position: relative;

    &::after {
      content: '';
      position: absolute;
      height: 0.3rem;
      width: 3rem;
      background-color: rgb(254, 55, 55);
      bottom: -0.5rem;
      left: 0;
    }
  }

  p {
    text-align: justify;
    text-justify: distribute;
  }

  span {
    color: red;
  }

  #carSelectGif {
    float: right;
    margin-top: 5%;
    width: 35%;
  }

  button {
    padding: 1.5rem 3rem;
    background-color: white;
    font-family: SilkScreen, Verdana, Geneva, Tahoma, sans-serif;
    font-size: 1.5rem;
    transition: transform 0.5s cubic-bezier(0.075, 0.82, 0.165, 1);
    border: 0.2rem solid black;
    float: right;
    clear: right;

    &:hover {
      transform: scale(1.05);
    }
  }
}

.inactive {
  opacity: 0.5 !important;
  pointer-events: none !important;

  *:hover {
    transform: none !important;
  }
}

h4 {
  margin-bottom: 2rem;
  color: grey;
}

.intersectionDisplay {
  width: 55rem;
  height: 35rem;
  overflow: hidden;
  margin-bottom: 1rem;
  border: 0.4rem solid black;
  padding: 2rem 4rem;
  background-color: #fff9f6;
}

.footer {
  position: relative;
  display: flex;
  flex-direction: column;
  width: 100%;
}

.result {
  position: absolute;
  left: 0;
  top: 0;
  max-width: 30%;
  font-size: 1.5rem;

  .correctResult {
    color: rgb(47, 205, 121);
  }

  .incorrectResult {
    color: rgb(240, 97, 41);
  }

  p {
    color: black;
    font-size: 1rem;
  }
}

.buttons {
  display: flex;
  align-self: flex-end;

  .buttons {
    button {
      padding: 1.5rem 3rem;
      background-color: white;
      font-family: SilkScreen, Verdana, Geneva, Tahoma, sans-serif;
    }

    #submitBtn {
      font-size: 2rem;
      transition: transform 0.5s cubic-bezier(0.075, 0.82, 0.165, 1);
      border: 0.2rem solid black;

      &:hover {
        transform: scale(1.05);
      }
    }

    #returnBtn {
      border: none;
      padding: 0 1rem;
      font-size: 1.5rem;
      color: grey;
      text-decoration: underline 2px rgba(128, 128, 128, 0);
      transition: text-decoration-color 0.5s cubic-bezier(0.075, 0.82, 0.165, 1);

      &:hover {
        text-decoration-color: rgba(128, 128, 128, 1);
      }
    }
  }
  gap: 1rem;

  button {
    padding: 1.5rem 3rem;
    background-color: white;
    font-family: SilkScreen, Verdana, Geneva, Tahoma, sans-serif;
  }

  #submitBtn {
    font-size: 2rem;
    transition: transform 0.5s cubic-bezier(0.075, 0.82, 0.165, 1);
    border: 0.2rem solid black;

    &:hover {
      transform: scale(1.05);
    }
  }

  #returnBtn {
    border: none;
    padding: 0 1rem;
    font-size: 1.5rem;
    color: grey;
    text-decoration: underline 2px rgba(128, 128, 128, 0);
    transition: text-decoration-color 0.5s cubic-bezier(0.075, 0.82, 0.165, 1);

    &:hover {
      text-decoration-color: rgba(128, 128, 128, 1);
    }
  }
}
#retryBtn {
  font-family: SilkScreen, Verdana, Geneva, Tahoma, sans-serif;
  color: grey;
  border: none;
  background-color: white;
  text-decoration: underline 2px rgb(128, 128, 128);
}
</style>
