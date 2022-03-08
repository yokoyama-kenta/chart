<template>
  <article class="chart-item">
    <p class="pl-1">推移</p>
    <div class="chart-wrapper">
      <LineChart
        class="line-chart"
        v-if="isLoaded"
        :chartData="{
          datasets,
          labels,
        }"
        :responseData="responseData"
        @sendLegend="setLegend"
      />
      <ul class="legend" v-html="chartLegend"></ul>
    </div>
  </article>
</template>

<script>
import axios from "axios";
import LineChart from "./Base/LineChart.vue";
import { addMonths, parse } from "date-fns"

export default {
  components: {
    LineChart,
  },
  data() {
    return {
      responseData: null,
      datasets: [],
      labels: [],
      isLoaded: false,
      chartLegend: null,
    };
  },
  async mounted() {
    await this.getData();
    this.isLoaded = true;
  },
  computed: {},
  methods: {
    async getData() {
      this.responseData = await axios
        .get("/mock/chart/transitionGraph.json")
        .then((response) => {
          return response.data;
        })
        .catch((error) => console.log(error));

      this.datasets = this.responseData.data.map((item) => {
        return {
          label: item.name,
          borderColor: item.color,
          backgroundColor: "transparent",
          pointBorderColor: "transparent",
          data: item.data,
          lineTension: 0,
          borderWidth: 1.5,
          pointRadius: 0,
          pointHitRadius: 0,
          hoverRadius: 0, // 丸ポチのサイズ
          pointBackgroundColor: item.color
        };
      });

      this.labels = this.responseData.data[0].data.map((item, index) => {
        return addMonths(parse(this.responseData.start, "yyyy-MM-dd", new Date()), index)
      });
    },
    setLegend (html) {
      this.chartLegend = html
    },
  },
};
</script>

<style scoped>
.chart-wrapper {
  display: flex;
}
.line-chart {
  flex-grow: 1;
  /* height: 100%; */
  position: relative;
  z-index: 0;
  background: #fff;
}
.legend {
  width: 100px;
  list-style: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-around;
  height: 180px;
}
.legend ::v-deep li {
  font-size: 12px;
  display: flex;
  align-items: center;
}
.legend ::v-deep .border {
  display: inline-block;
  width: 10px;
  height: 2px;
  margin-right: 6px;
}
</style>
