<script>
import { ref, onMounted} from 'vue';

export default {
  props : {
    floor : Number
  },
  setup(props) {
    const elev = ref(null);
    return { elev };
  }
}
</script>

<template>
    <div class="elevator" ref="elev"
    :style="{
      bottom : ((floor - 1) * 120).toString() + 'px'
    }">
      <div class="display">{{floor.toString()}}</div>
      <div class="display"></div>
    </div>
</template>

<style>

.elevator {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  background-color: aqua;
  width: 100%;
  height: 120px;
  position: absolute;
}

.display {
  text-align: center;
  width : 50px;
  height : 20px;
  border: solid black;
  background-color: white;
  color : 'red';
}

</style>