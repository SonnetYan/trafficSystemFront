<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <TrafficAnimation/>
</template>

<script>
import TrafficAnimation from './components/TrafficAnimation.vue'

export default {
  name: 'App',
  components: {
   // HelloWorld
   TrafficAnimation
   
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #d8dadc;
  margin-top: 60px;
}
</style>
