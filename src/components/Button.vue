<script>
export default {
  props: {
    number: Number
  }
}
</script>

<template>
  <div class="buttonContainer">
    <div class="buttonLabel">
      {{number}}
    </div>
    <div class="button">
      <input type="radio" checked>
    </div>
  </div>
</template>

<style scoped>

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