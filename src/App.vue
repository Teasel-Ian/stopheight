<template>
  <img alt="Vue logo" src="./assets/teasel_avatar.jpg" />
  <StopHeight chain="fetchhub" />
</template>

<script>
import StopHeight from './components/StopHeight.vue'

export default {
  name: 'App',
  components: {
    StopHeight,
  },
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
