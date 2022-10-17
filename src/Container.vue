<script setup>
  import {ref, onMounted} from 'vue';
  import Button from './components/Button.vue';
  import Elevator from './components/Elevator.vue';

  let elevatorMap = JSON.parse(sessionStorage.getItem('elevatorMap'));

  let liftCount = null;
  let floorsCount = null;
  let floors = null;
  let elevators = null;
  let eelevators = null;
  let callQueue = [];
  let callQueuePromise = null;

  if (elevatorMap) {
    liftCount = ref(elevatorMap.liftCount);
    floorsCount = ref(elevatorMap.floorsCount);

    // keys of floors and elevators for "v-for" correct applying
    floors = ref(elevatorMap.floors);
    elevators = ref(elevatorMap.elevators);

    // state for all elevators
    eelevators = elevatorMap.eelevators;
  }
  else {
    // number of elevators and floors for map size defenition
    liftCount = ref(1);
    floorsCount = ref(5);

    // keys of floors and elevators for "v-for" correct applying
    floors = ref(Array.from(Array(floorsCount.value).keys()).map(item=>(item + 1)));
    elevators = ref(Array.from(Array(liftCount.value).keys()));

    // state for all elevators
    eelevators = Array.from(Array(liftCount.value).keys()).map((__, index)=>({
      floor : 1,
      previousFloor : 1,
      isBusy : false,
      action : null,
      index
    }));
  }

  // refs for refering to elevator's DOM nodes
  const reff = ref(null);
  const refff = ref(null);

  function floorsCountChanging(e) {
    floorsCount.value = parseInt(e.target.value);
    floors.value = Array.from(Array(parseInt(e.target.value)).keys()).map(item=>(item + 1));
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
  
  function numberOfFloor(count, display, delta) {
    display.textContent = (parseInt(display.textContent) + delta).toString();
    if (count) setTimeout(()=>numberOfFloor(count - 1, display, delta), 1000);
  }
  
  const elevatorMove = async (index, newFloor)=>{
    
      let previousFloor = eelevators[index].floor
      eelevators[index].isBusy = true;
      
      // draws floor's number and direction
      let displays = reff.value[index].elev.querySelectorAll('.display');
      let delta = Math.sign(newFloor - previousFloor);
      let count = Math.abs(newFloor - previousFloor);
      if (delta < 0) displays[1].textContent = 'down';
      else displays[1].textContent = 'up';
      setTimeout(()=>numberOfFloor(count - 1, displays[0], delta), 1000);

      // transition animation
      await reff.value[index].elev.animate([
        {transform : 'translateY(0)'},
        {transform : 'translateY(' + (-120 * (newFloor - previousFloor)).toString() + 'px)'}
      ], {duration: 1000 * Math.abs(newFloor - previousFloor)}).finished;
      reff.value[index].elev.style.bottom = (120 * (newFloor - 1)).toString() + 'px';
      displays[1].textContent = '';
      eelevators[index].previousFloor = eelevators[index].floor;
      eelevators[index].floor = newFloor;

      // flickering animation
      await reff.value[index].elev.animate([
        {opacity : 1},
        {opacity : 0},
        {opacity : 1}
      ], {duration: 1000,
          iterations : 3}).finished;

      eelevators[index].isBusy = false;
      refff.value[newFloor - 1].button.style.backgroundColor = 'white';
      callQueue.shift();
      return index;
      };

  function buttonClicked(targetFloor) {
    if (!callQueue.includes(targetFloor)) {
      let freeElevators = eelevators.filter(item=>!item.isBusy);
      refff.value[targetFloor - 1].button.style.backgroundColor = 'red';
      // if all elevators are busy
      if (!freeElevators.length) {
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
          callQueue.push(targetFloor);
          nearestElevator.action = elevatorMove(nearestElevator.index, targetFloor);
        }
      }
    }
  }
  
  // prevents changes of map by keyboard input
  function keydownHandler(e) {
    if (e.code !== 'ArrowUp' && e.code !== 'ArrowDown') e.preventDefault();
  }

  // resume animations from last positions
  onMounted(()=>{
    if (elevatorMap) {
      for (let x of elevatorMap.callQueue) {
        buttonClicked(x);
      }
    }
  });

  window.onbeforeunload = (event)=> {
    eelevators.forEach(item=>{item.isBusy = false; item.action = null})
    sessionStorage.setItem('elevatorMap', JSON.stringify({
      liftCount : liftCount.value,
      floorsCount : floorsCount.value,
      elevators : elevators.value,
      floors : floors.value,
      eelevators,
      callQueue
    })
    )
  };
  
</script>

<template>
  <header id="header">
    <span>
    Select number of lifts
    </span>
    <input @keydown="keydownHandler" @change="columnsCountChanging" class="inputNumber" type="number" :value="liftCount" min="1"/>
    <span>
    Select number of floors
    </span>
    <input @keydown="keydownHandler" @change="floorsCountChanging" class="inputNumber" type="number" :value="floorsCount" min="2"/>
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
      <div class="column" v-for="elevator in elevators" :key="elevator">
        <Elevator ref="reff" :floor="eelevators[elevator].floor"/>
      </div>
      <div class="buttons">
        <Button ref="refff" v-for="floor in floors" :key="floor" :number="floor" @buttonClicked="buttonClicked"/>
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
  display: flex;
  flex-direction: column-reverse;
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
  border: solid rgb(99, 99, 99) 3px;
  display: flex;
  flex-direction: row;
  justify-content: start;
}

</style>