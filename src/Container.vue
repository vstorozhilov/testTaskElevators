<script setup>
  import {ref, onMounted} from 'vue';
  import Button from './components/Button.vue';
  import Elevator from './components/Elevator.vue';

  const liftCount = ref(1);
  const floorsCount = ref(5);
  const floors = ref(Array.from(Array(floorsCount.value).keys()).map(item=>(item + 1)).reverse());
  const elevators = ref(Array.from(Array(liftCount.value).keys()).map((item, index)=>({
    floor : 1,
    isBusy : false,
    previousFloor : 1,
    index
  })));

  const reff = ref(null);

  //onMounted(setTimeout(()=>console.log(reff.value), 5000))

  //const elevs = Array.from(Array(liftCount.value).keys()).map(item=>ref(null));

  function floorsCountChanging(e) {
    floorsCount.value = e.target.value;
    floors.value = Array.from(Array(parseInt(e.target.value)).keys()).map(item=>(item + 1)).reverse();
  }

  function columnsCountChanging(e) {
    liftCount.value = e.target.value;
    elevators.value = Array.from(Array(parseInt(e.target.value)).keys()).map((item, index)=>({
      floor : 1,
      isBusy : false,
      previousFloor : 1,
      index
    }));
  }

</script>

<template>
  <header id="header">
    <span>
    Select number of lifts
    </span>
    <input @change="columnsCountChanging" class="inputNumber" type="number" value="1" min="1"/>
    <span>
    Select number of floors
    </span>
    <input @change="floorsCountChanging" class="inputNumber" type="number" value="5" min="2"/>
  </header>
  <main id="main">
    <div id="building" :style="{
      height : (floorsCount * 120).toString() + 'px',
      width : (liftCount <= 9 ? '1100px' : (liftCount * 110 + 10 * (liftCount - 1)).toString() + 'px')
      }">
      <div id="background">
        <div class="floor" v-for="floor in floors" :key="floor">
        </div>
        </div>
      <div class="column" v-for="elevator in elevators" :key="elevator.index">
        <Elevator :elevator="elevator" ref="reff"/>
      </div>
      <div class="buttons" @buttonClicked="()=>{}">
        <Button  v-for="floor in floors" :key="floor" :number="floor"/>
      </div>
    </div>
  </main>
</template>

<style>

* {
  box-sizing: border-box;
}

.buttons {
  height: 100%;
  width: 50px;
  /* border: solid black 2px; */
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.floor {
  height: 100%;
  border-bottom: solid black 2px;
}

#background {
  display: flex;
  flex-direction: column;
  position : absolute;
  flex-grow: 1;
  width : 100%;
  height : 100%;
  z-index: -1;
}

.column {
  display: flex;
  position: relative;
  flex-direction: column;
  justify-content: flex-end;
  height: 100%;
  width: 110px;
  border-right: solid rgb(99, 92, 92) 3px;
  border-left: solid rgb(99, 92, 92) 3px;
}

::-webkit-scrollbar {
  display: none
}

.inputNumber {
  width : 40px;
}

#header {
  display: flex;
  justify-content: center;
  gap: 10px;
}

#main {
  margin-top: 10px;
  display: flex;
  justify-content: center;
  width: 100%;
}

#building {
  overflow-x: scroll;
  overflow-y: scroll;
  width: 1100px;
  gap : 30px;
  position : relative;
  /* height : 700px; */
  border: solid rgb(99, 99, 99) 3px;
  display: flex;
  flex-direction: row;
  justify-content: start;
}

</style>