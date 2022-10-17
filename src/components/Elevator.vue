<script>
import { ref, onMounted} from 'vue';

export default {
  setup(props) {
    const elev = ref(null);
    return { elev };
  }
}
</script>

<template>
    <div class="elevator" ref="elev">
    </div>
</template>

<style>

.elevator {
  background-color: aqua;
  width: 100%;
  height: 120px;
  position: absolute;
  bottom: 0px;
}

</style>