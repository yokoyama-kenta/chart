<template>
  <div class="card">
    上位グラフ
    <HorizontalBarChart
      v-if="isLoaded"
      :chartData="{
        datasets,
        labels,
      }"
      :options="options"
    />
  </div>
</template>

<script>
import axios from "axios";
import HorizontalBarChart from "./Base/HorizontalBarChart.vue";

export default {
  components: {
    HorizontalBarChart,
  },
  data() {
    return {
      datasets: [],
      labels: [],
      options: {},
      isLoaded: false,
    };
  },
  async mounted() {
    await this.getData();
    this.isLoaded = true;
  },
  computed: {},
  methods: {
    async getData() {
      // 月ごとにリクエスト
      const chartData = await axios
        .get("/mock/chart/topGraph.json")
        .then((response) => {
          return response.data
        })
        .catch((error) => console.log(error));

      this.datasets = [
        {
          label: "売上",
          data: chartData.map((item) => {
            return item.profit;
          }),
          backgroundColor: "#0EBAB6",
          barPercentage: 0.3,
          maxBarThickness: 30,
        },
      ];

      this.labels = chartData.map((item) => {
        return item.branch_name;
      });

      this.options = {
        maintainAspectRatio: false,
        responsive: true,
        scales: {
          xAxes: [
            {
              gridLines: {
                color: "transparent",
                zeroLineColor: "#E6E6E6",
                drawTicks: false,
              },
              ticks: {
                beginAtZero: true,
                callback: (value, index) => {
                  return index === 0 ? `${value}万円` : value.toLocaleString();
                },
                fontColor: "#848484",
                padding: 10,
              },
            },
          ],
          yAxes: [
            {
              gridLines: {
                color: "transparent",
                drawOnChartArea: false,
                drawTicks: false,
              },
              ticks: {
                callback: (value, index) => {
                  value = `${chartData[index].order}：${value}`
                  return value.length > 8 ? value.substr(0, 7) + "…" : value
                },
                fontColor: "#848484",
                padding: 15,
              },
            },
            {
              gridLines: {
                color: "transparent",
                zeroLineColor: "#E6E6E6",
                drawTicks: false,
              },
              ticks: {
                display: false,
              },
            },
          ],
        },
        legend: {
          display: false,
        },
        tooltips: {
          mode: "label", //マウスオーバー時に表示されるtooltip
        },
      };
    },
  },
};
</script>

<style>
.card {
  width: 800px;
  height: 400px;
}
</style>
