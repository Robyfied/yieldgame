<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  type: {
    type: String,
    required: true,
    validator(value) {
      return value.length == 1 && '+T'.includes(value)
    }
  },
  structure: {
    type: String,
    validator(value, props) {
      return (
        value.split('').every((char) => 'RLTB'.includes(char)) && // Right, Left, Top, Bottom
        (props.type == 'T' ? value.length == 3 : value.length == 4)
      )
    },
    default: (props) => (props.type == 'T' ? 'LBR' : 'RLTB')
  },
  priorityRoad: {
    type: String,
    required: true,
    validator(value, props) {
      return value.split('').every((char) => props.structure.includes(char)) && value.length == 2
    }
  },
  cars: {
    type: String,
    validator(value, props) {
      return value.split('').every((char) => props.structure.includes(char))
    }
  },
  selected: {
    type: String,
    default: '',
    validator(value, props) {
      return value.split('').every((char) => props.cars.includes(char))
    }
  },
  stopSigns: {
    type: String,
    validator(value, props) {
      return value.split('').every((char) => !props.priorityRoad.includes(char))
    }
  },
  playerCar: {
    type: String,
    validator(value) {
      return '1234'.includes(value)
    }
  },
  direction: {
    type: String,
    validator(value, props) {
      return props.structure.includes(value) && value != 'B'
    }
  }
})

const emit = defineEmits(['carSelect'])

const getRandInt = (max) => Math.floor(Math.random() * (max + 1))

const getCarTextures = () => {
  let npcCars = ['1', '2', '3', '4'].filter((e) => e != props.playerCar)
  let returnVal = {}
  for (let i = 0; i < props.cars.length; i++) {
    const roadPosition = props.cars[i]
    if (roadPosition == 'B')
      returnVal['B'] = `/car_textures/${props.playerCar}/car${props.playerCar}B.png`
    else {
      const carNo = npcCars[getRandInt(npcCars.length - 1)]
      npcCars = npcCars.filter((e) => e != carNo)
      returnVal[props.cars[i]] = `/car_textures/${carNo}/car${carNo}${roadPosition}.png`
    }
  }
  return returnVal
}

const roads = computed(() => {
  const carTextures = ref(getCarTextures())
  return [...Array(props.structure.length)].map((_, ix) => {
    return {
      position: props.structure[ix],
      hasPriority: props.priorityRoad.includes(props.structure[ix]),
      hasStop: props.stopSigns != null ? props.stopSigns.includes(props.structure[ix]) : false,
      hasCar: props.cars.includes(props.structure[ix]),
      carTex: carTextures.value[props.structure[ix]],
      selected: false
    }
  })
})

const selectedCars = ref(props.selected.split(''))
const selectCar = (road) => {
  if (road.position !== 'B') {
    roads.value.filter((e) => e.position == road.position)[0].selected = !roads.value.filter(
      (e) => e.position == road.position
    )[0].selected
    selectedCars.value = roads.value.filter((e) => e.selected == true).map((e) => e.position)

    emit('carSelect', selectedCars)
  }
}

const getAdditionalPannel = (intersectionType, priorityRoad, intersectionStructure) => {
  if (priorityRoad[0] != 'L' && priorityRoad[0] != 'R')
    priorityRoad = priorityRoad.split('').reverse().join('')
  if (intersectionType == '+') return `/sign_textures/additional/+/${priorityRoad}+.png`
  else if (priorityRoad.includes('T')) return `/sign_textures/additional/T/${priorityRoad}T.png`
  else if (intersectionStructure.includes('T'))
    return `/sign_textures/additional/T/${priorityRoad}Tv.png`
  else return `/sign_textures/additional/T/${priorityRoad}Th.png`
}

const getRoadTexture = (road) => {
  if (road.hasPriority) return '/road_textures/roadPriority.png'
  if (road.hasStop) return '/road_textures/roadStop.png'
  return '/road_textures/roadYield.png'
}

const getSignTexture = (road) => {
  if (road.hasPriority) return '/sign_textures/priority.png'
  if (road.hasStop) return '/sign_textures/stop.png'
  return '/sign_textures/yield.png'
}

const missingRoad = computed(() => {
  for (let road of ['L', 'R', 'T']) if (!props.structure.includes(road)) return road
  return 'ERR'
})

const arrowColorFilter = computed(() => {
  switch (props.playerCar) {
    case '1':
      return 'hue-rotate(180deg)'
    case '2':
      return 'hue-rotate(0) opacity(0.8)'
    case '3':
      return 'saturate(0) brightness(3)'
    case '4':
      return 'hue-rotate(193deg) brightness(1.5) opacity(0.7)'
  }
  return 'hue-rotate(0)'
})

const positionToDegrees = {
  R: 0,
  B: 90,
  L: 180,
  T: 270
}
</script>

<template>
  <div class="wrapper">
    <div class="intersection">
      <div v-if="props.type === 'T'" :class="`TIntersectionLine ${missingRoad}RoadLine`"></div>
      <img
        :src="`/road_textures/turn${props.direction}.png`"
        :id="`directionArrow${props.direction}`"
        alt=""
        class="directionArrow"
        draggable="false"
      />
      <div
        v-for="road in roads"
        :key="road.position"
        class="road"
        :style="{ transform: `rotate(${positionToDegrees[road.position]}deg)` }"
        draggable="false"
      >
        <img class="roadTex" :src="getRoadTexture(road)" alt="roadTex" draggable="false" />
        <div class="signWrapper">
          <img
            class="sign"
            :id="`sign${road.position}`"
            :src="getSignTexture(road)"
            draggable="false"
          />
          <img
            class="sign signBack"
            :id="`sign${road.position}`"
            :src="getSignTexture(road).replace('.png', '_back.png ')"
            draggable="false"
          />
          <img
            v-if="!['RL', 'LR', 'TB', 'BT'].includes(props.priorityRoad)"
            class="additionalPanel"
            :id="`additional${road.position}`"
            :src="getAdditionalPannel(props.type, props.priorityRoad, props.structure)"
            alt="additionalPanel"
            draggable="false"
          />
          <img
            v-if="!['RL', 'LR', 'TB', 'BT'].includes(props.priorityRoad)"
            class="additionalPanel additionalBack"
            src="/sign_textures/additional_back.png"
            alt="backside"
            draggable="false"
          />
        </div>
        <img
          v-if="road.hasCar"
          class="car"
          :class="{
            selected: selectedCars.includes(road.position),
            pointerCursor: road.position != 'B'
          }"
          :id="`car${road.position}`"
          :src="road.carTex"
          @click="selectCar(road)"
          alt="carTex"
          draggable="false"
        />
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.wrapper {
  perspective: 600px;
  box-sizing: content-box;
}

.intersection {
  position: relative;
  display: flex;
  align-items: center;
  height: inherit;
  transform: rotateX(65deg);
  transform-style: preserve-3d;
  image-rendering: pixelated;
}

.road {
  display: flex;
  position: absolute;
  align-items: center;
  justify-content: center;
  padding: 0;
  margin: 0;
  left: 50%;
  transform-origin: center left;
  transform-style: preserve-3d;
  width: 40%;

  .roadTex {
    padding: 0;
    margin: 0;
    width: 100%;
  }
}

.directionArrow {
  position: absolute;
  filter: v-bind(arrowColorFilter);
  transform-origin: center bottom;
  transform: translateZ(1px);
  transform-style: preserve-3d;
  &#directionArrowT {
    bottom: 25%;
    left: 51%;
    height: 50%;
  }
  &#directionArrowR {
    left: 53%;
    bottom: 25%;
    height: 25%;
  }
  &#directionArrowL {
    bottom: 24%;
    left: 32.5%;
    height: 40%;
  }
}

.signWrapper {
  position: absolute;
  height: 70%;
  width: 13%;
  display: flex;
  justify-content: center;
  top: -80%;
  left: 30%;
  rotate: -90deg;
  transform: rotateX(-90deg);
  transform-origin: bottom center;
  transform-style: preserve-3d;

  &::before {
    content: '';
    width: 120%;
    height: 80%;
    background-color: green;
    position: absolute;
    bottom: 0;
    transform: rotateX(90deg) translateZ(-3.2em);
    transform-style: preserve-3d;
  }

  .sign {
    position: absolute;
    height: 100%;
  }

  .signBack {
    transform: translateZ(-0.3px);
  }

  .additionalPanel {
    position: absolute;
    height: 25%;
    top: 35%;

    &#additionalR {
      rotate: 90deg;
    }
    &#additionalT {
      rotate: 180deg;
    }
    &#additionalL {
      rotate: 270deg;
    }
    &.additionalBack {
      background-color: #5b5b5b;
      transform: translateZ(-0.1px);
    }
  }
}

.pointerCursor {
  cursor: pointer;
}

.car {
  position: absolute;
  transform-origin: center bottom;
  transform: rotateX(-55deg);
  transform-style: preserve-3d;
  z-index: 5;
  filter: drop-shadow(0px 5px 2px rgba(0, 0, 0, 0.4)) drop-shadow(rgba(255, 255, 255, 1));

  &#carR {
    width: 50%;
    left: 30%;
    top: 5%;
  }
  &#carT {
    rotate: 90deg;
    width: 23%;
    left: 25%;
    top: -8%;
  }
  &#carL {
    rotate: 180deg;
    width: 50%;
    left: 30%;
    top: -20%;
  }
  &#carB {
    rotate: 270deg;
    width: 23%;
    left: 52%;
    top: -10%;
  }
}

.selected {
  filter: sepia(100%) hue-rotate(-40deg) saturate(150%) drop-shadow(4px 0 0 rgb(255, 0, 0))
    drop-shadow(-4px 0 0 rgb(255, 0, 0)) drop-shadow(0 4px 0 rgb(255, 0, 0))
    drop-shadow(0 -4px 0 rgb(255, 0, 0));
}

.TIntersectionLine {
  position: absolute;
  transform: translateY(-50%);
  height: 100%;
  width: 4px;
  background-color: white;
  z-index: 5;
  &.LRoadLine {
    left: 40%;
    top: 50%;
  }
  &.RRoadLine {
    right: 40%;
    top: 50%;
  }
  &.TRoadLine {
    rotate: 90deg;
    top: -15%;
  }
}
</style>
