<script setup>
import { ref } from 'vue'

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
  hasLights: {
    type: Boolean,
    default: false
  },
  cars: {
    type: String,
    validator(value, props) {
      return value.split('').every((char) => props.structure.includes(char))
    }
  },
  stopSigns: {
    type: String,
    validator(value, props) {
      return value.split('').every((char) => !props.priorityRoad.includes(char))
    }
  }
})

const roads = ref(
  [...Array(props.structure.length)].map((_, ix) => {
    return {
      position: props.structure[ix],
      hasPriority: props.priorityRoad.includes(props.structure[ix]),
      hasStop: props.stopSigns != null ? props.stopSigns.includes(props.structure[ix]) : false,
      hasLight: props.hasLights,
      lightOn: false,
      hasCar: props.cars.includes(props.structure[ix]),
      selected: false
    }
  })
)

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
      <div
        v-for="road in roads"
        :key="road.position"
        class="road"
        :id="'road' + road.position"
        :style="{ transform: 'rotate(' + positionToDegrees[road.position] + 'deg)' }"
      >
        <img
          class="roadTex"
          v-if="road.hasPriority"
          src="/road_textures/roadPriority.png"
          alt="roadTex"
        />
        <img
          class="roadTex"
          v-else-if="road.hasStop"
          src="/road_textures/roadStop.png"
          alt="roadTex"
          style="z-index: 2"
        />
        <img
          class="roadTex"
          v-else
          src="/road_textures/roadYield.png"
          alt="roadTex"
          style="z-index: 3"
        />
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.wrapper {
  perspective: 400px;
}

.intersection {
  position: relative;
  display: flex;
  justify-content: center;
  align-content: center;
  align-items: center;
  width: inherit;
  height: inherit;
  transform: rotateX(40deg);
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
  width: 40%;
  perspective: 100px;

  &::after {
    content: '';

    background-repeat: no-repeat;
    background-size: contain;
    // background-color: red;
    position: absolute;
    transform-origin: center center;
    transform: rotateX(-20deg);
    width: 80%;
    height: 80%;
    z-index: 10;
  }

  img {
    padding: 0;
    margin: 0;
    width: 100%;
  }
}

#roadR::after {
  background-image: url('/car_textures/carR.png');
  left: 20%;
  top: -35%;
}

#roadT::after {
  background-image: url('/car_textures/carT.png');
  rotate: 90deg;
  left: 5%;
  top: 0;
}

#roadL::after {
  background-image: url('/public/car_textures/carL.png');
  rotate: 180deg;
}

#roadB::after {
  background-image: url('/public/car_textures/carB.png');
  rotate: 270deg;
}
</style>
