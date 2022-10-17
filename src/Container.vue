<script setup>
  import {ref, onMounted} from 'vue';
  import Button from './components/Button.vue';
  import Elevator from './components/Elevator.vue';

  // number of elevators and floors for map size defenition
  const liftCount = ref(1);
  const floorsCount = ref(5);

  // keys of floors and elevators for "v-for" correct applying
  const floors = ref(Array.from(Array(floorsCount.value).keys()).map(item=>(item + 1)).reverse());
  const elevators = ref(Array.from(Array(liftCount.value).keys()));

  // state for all elevators
  let eelevators = Array.from(Array(liftCount.value).keys()).map((__, index)=>({
    floor : 1,
    previousFloor : 1,
    isBusy : false,
    action : null,
    index
  }));

  let callQueuePromise = null;
  let callQueue = [];
  // refs for refering to elevator's DOM nodes
  const reff = ref(null);

  function floorsCountChanging(e) {
    floorsCount.value = e.target.value;
    floors.value = Array.from(Array(parseInt(e.target.value)).keys()).map(item=>(item + 1)).reverse();
  }

  function columnsCountChanging(e) {
    let newColumnValue = parseInt(e.target.value);
    liftCount.value = newColumnValue;
    elevators.value = Array.from(Array(newColumnValue).keys());

    if (eelevators.length < newColumnValue) eelevators.push({
      floor : 1,
      previousFloor : 1,
      isBusy : false,
      action : null,
      index : parseInt(e.target.value) - 1
    })
    else eelevators.pop();
  }

  const elevatorMove = async (index, newFloor)=>{

      eelevators[index].previousFloor = eelevators[index].floor;
      eelevators[index].floor = newFloor;
      eelevators[index].isBusy = true;

      await reff.value[index].elev.animate([
        {transform : 'translateY(0)'},
        {transform : 'translateY(' + (-120 * (newFloor - eelevators[index].previousFloor)).toString() + 'px)'}
      ], {duration: 1000 * Math.abs(newFloor - eelevators[index].previousFloor)}).finished;
      reff.value[index].elev.style.bottom = (120 * (newFloor - 1)).toString() + 'px';
      await reff.value[index].elev.animate([
        {opacity : 1},
        {opacity : 0},
        {opacity : 1}
      ], {duration: 1000,
          iterations : 3}).finished;
      eelevators[index].isBusy = false;
      callQueue.shift();
      return index;
      };

  function buttonClicked(targetFloor) {
    let freeElevators = eelevators.filter(item=>!item.isBusy);

    // if all elevators are busy
    if (!freeElevators.length) {
      if (!callQueue.includes(targetFloor)) {
        callQueue.push(targetFloor)
        if (!callQueuePromise) {
          callQueuePromise =  Promise.any(eelevators.map(item=>item.action)).then(index=>{
            let newCall = elevatorMove(index, targetFloor);
            eelevators[index].action = newCall;
          });
        }
        else {
          callQueuePromise = callQueuePromise.then(()=>{
              return Promise.any(eelevators.map(item=>item.action)).then(index=>{
                let newCall = elevatorMove(index, targetFloor);
                eelevators[index].action = newCall;
              })
            }
          )
        }
      }
      // let fastestIndex;
      // let newcall = Promise.any(eelevators.map(item=>item.action)).then(index=>{
      //   fastestIndex = index;
      //   // let newcall = elevatorMove(index, targetFloor);
      //   // eelevators[index].action = newcall;
      //   return elevatorMove(index, targetFloor);
      // });
      // eelevators[fastestIndex].action = newcall;
    }
    else {
      let min = floorsCount.value;
      let nearestElevator = null;
      for (let elevator of freeElevators) {
        if (Math.abs(elevator.floor - targetFloor) < min) {
          min = Math.abs(elevator.floor - targetFloor);
          nearestElevator = elevator;
        }
      }
      // if the nearest elevator isn't on the same floor
      if (min) {
        nearestElevator.action = elevatorMove(nearestElevator.index, targetFloor);
      }
    }
  }

  // prevents changes of map by keyboard input
  function keydownHandler(e) {
    if (e.code !== 'ArrowUp' && e.code !== 'ArrowDown') e.preventDefault();
  }

</script>

<template>
  <header id="header">
    <span>
    Select number of lifts
    </span>
    <input @keydown="keydownHandler" @change="columnsCountChanging" class="inputNumber" type="number" value="1" min="1"/>
    <span>
    Select number of floors
    </span>
    <input @keydown="keydownHandler" @change="floorsCountChanging" class="inputNumber" type="number" value="5" min="2"/>
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
        <Elevator ref="reff"/>
      </div>
      <div class="buttons">
        <Button  v-for="floor in floors" :key="floor" :number="floor" @buttonClicked="buttonClicked"/>
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
  overflow-x: scroll;
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