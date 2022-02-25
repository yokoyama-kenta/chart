<template>
  <div id="app">
    <TopGraph class="topGraph" />
    <Transition class="transition" />
  </div>
</template>

<script>
import TopGraph from './components/Chart/TopGraph.vue'
import Transition from './components/Chart/Transition.vue'

export default {
  name: 'App',
  components: {
    TopGraph,
    Transition
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
.topGraph {
  margin-bottom: 100px;
}
.transition {
}
</style>
