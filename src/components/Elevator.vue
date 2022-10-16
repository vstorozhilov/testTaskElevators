<script>
import { ref, onMounted} from 'vue';

export default {
  setup(props) {
    const elev = ref(null);
    // onMounted(async ()=>{
    //   await elev.value.animate([
    //     {transform : 'translateY(0)'},
    //     {transform : 'translateY(-100%)'}
    //   ], {duration: 1000}).finished;
    //   elev.value.style.bottom = '120px'
    //   await elev.value.animate([
    //     {opacity : 1},
    //     {opacity : 0},
    //     {opacity : 1}
    //   ], {duration: 1000,
    //       iterations : 3}).finished;
    //   console.log('Mouned');
    //   });
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