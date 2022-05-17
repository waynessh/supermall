<template>
  <div class="check-button" :class="{check:isChecked}">
    <img src="../../../assets/img/cart/tick.svg" alt="">
  </div>
</template>

<script>
export default {
  name:"CheckButton",
  props:{
    isChecked:{
      type:Boolean,
      default:false
    }
  }
}
</script>

<style scoped>
  .check-button img{
    border: 1px solid #aaa;
    align-items: center;
    height: 20px;
    width: 20px;
    justify-content: center;
    border-radius: 50%;
  }
   .check {
    background-color: var(--color-high-text);
    border-radius: 50%;
  }
</style>