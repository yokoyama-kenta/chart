<template>
  <div id="app">
    <div class="chart">
      <h2 class="chart-title">チャートサンプル</h2>
    </div>
    <ul class="chart-list">
      <li class="chart-list__item top">
        <TopGraph />
      </li>
      <li class="chart-list__item transition">
        <Transition />
      </li>
    </ul>
    <DashBoard class="dashBoard" />
  </div>
</template>

<script>
import TopGraph from './components/Chart/TopGraph.vue'
import Transition from './components/Chart/Transition.vue'
import DashBoard from './components/Chart/DashBoard.vue'

export default {
  name: 'App',
  components: {
    TopGraph,
    Transition,
    DashBoard
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
.chart {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 40px;
}

.chart-title {
  font-size: 16px;
  font-weight: 400;
}

.chart-list {
  list-style: none;
  margin-top: 15px;
  display: flex;
}
.chart-list__item {

}
.top {
  width: 35%;
  padding-right: 15px;
}
.transition {
  width: 65%;
  padding-left: 15px;
}
</style>
