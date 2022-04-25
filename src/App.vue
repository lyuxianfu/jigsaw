<template>
  <div @click="visible = !visible">验证</div>
  <Jigsaw v-if="visible" @close="handleClose" @check="handleCheck"></Jigsaw>
</template>

<script lang="ts" setup>
import {ref} from 'vue'
import Jigsaw from './components/Jigsaw.vue';
const visible =ref(false)
const handleClose = ():void=>{
  visible.value = false
}
const handleCheck = (check: boolean):void=>{
  alert('验证'+check)
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
