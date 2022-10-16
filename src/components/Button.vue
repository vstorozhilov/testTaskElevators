<script>
export default {
  props: {
    number: Number
  },
  setup (props, context) {
    const handleClick = () => {
        context.emit("buttonClicked", props.number)
    }
    return {
        handleClick
    }
  },
}
</script>

<template>
  <div class="buttonContainer">
    <div class="buttonLabel">
      {{number}}
    </div>
    <div class="button"
      @mousedown="(e)=>{e.currentTarget.style.backgroundColor = 'aqua'}"
      @mouseup="(e)=>{e.currentTarget.style.backgroundColor = 'transparent'}"
      @click="handleClick">
      <input type="radio" checked>
    </div>
  </div>
</template>

<style>

.buttonLabel {
  width: 30px;
  height: 30px;
}
.buttonContainer {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  height: 120px;
  width : 100%;
}

.button {
  display: flex;
  justify-content: center;
  align-items: center;
  width : 30px;
  height: 30px;
  border: solid black 1px;
}

</style>