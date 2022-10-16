<script>
import { ref, onMounted } from 'vue';
import {Keyframes} from 'vue-keyframes';

export default {
  props: {
    elevator: Object,
    id : Number
  },
  setup(props) {
    console.log(props.elevator);
    const elev = ref(null);
    onMounted(async ()=>{
      //let elevator = document.querySelector('#' + 'elevator' + (props.elevator.index).toString());
      await elev.value.animate([
        {transform : 'translateY(0)'},
        {transform : 'translateY(-100%)'}
      ], {duration: 1000}).finished;
      elev.value.style.bottom = '120px'
      await elev.value.animate([
        {opacity : 1},
        {opacity : 0},
        {opacity : 1}
      ], {duration: 1000,
          iterations : 3}).finished;
      //elevator.style.bottom = '120px'
      console.log('Mouned');
      });
    return { elev };
  }
}
</script>

<template>
    <div class="elevator" ref="elev" :id="'elevator' + elevator.index"
    :style="{
      //bottom: '120px'
      //bottom: '120px',
      // animationName: 'transition',
      // animationDuration: '3s',
    }">
    </div>
</template>

<style>

@keyframes transition {
  from {
    transform: translateY(0);
  }

  to {
    transform: translateY(-100%);
  }
}

@keyframes flickering {
  from {
    opacity: 1;
  }

  50% {
    opacity : 0;
  }

  to {
    opacity : 1;
  }
}

.elevator {
  /* animation-duration: 1s;
  animation-name: flickering;
  animation-timing-function: linear;
  animation-iteration-count: 3; */
  background-color: aqua;
  width: 100%;
  height: 120px;
  position: absolute;
  bottom: 0px;
  /* border: solid black 3px;
  margin-left: 0px; */

}

</style>