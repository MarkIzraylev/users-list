<script setup>
  import {defineModel} from 'vue'
  
  const modelValue = defineModel();
</script>

<template>
  <input class="input" v-model="modelValue" />
</template>

<style scoped>
  .input {
    background-image: url('../assets/search.svg');
    background-size: 24px;
    background-position: 3px center;
    background-repeat: no-repeat;
    padding-left: 31px;
    padding-right: 9px;
    line-height: 16px;
    width: 100%;
    height: 34px;
    outline: none;
    border: none;
    background-color: #ECEFF0;
    border-radius: 4px;
    color: #333333;
    font-size: 12px;
    font-weight: 500;
  }
  .input::placeholder {
    color: #9EAAB4;
  }
</style>