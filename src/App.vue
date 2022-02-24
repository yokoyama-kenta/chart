<template>
  <div id="app">
    <TopGraph />
  </div>
</template>

<script>
import TopGraph from './components/Chart/TopGraph.vue'

export default {
  name: 'App',
  components: {
    TopGraph
  }
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
