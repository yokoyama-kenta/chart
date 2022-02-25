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
      isLoaded: false,
    };
  },
  async mounted() {
    await this.getData();
    this.isLoaded = true;
  },
  computed: {
    options() {
      return {
        maintainAspectRatio: false,
        responsive: true,
        scales: {
          xAxes: [
            {
              gridLines: {
                color: "transparent",
                zeroLineColor: "#E6E6E6",
                // display: false,
                drawTicks: false,
                // drawOnChartArea: false
                // drawBorder: false,
                // display: false,
                // display: false,
                // drawTicks: true,
                // drawBorder: false,
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
              maxBarThickness: 30,
              gridLines: {
                color: "transparent",
                // zeroLineColor: "#E6E6E6",
                drawOnChartArea: false,
                // display: false,
                // drawOnChartArea: false
                drawTicks: false,
                // display: false,
                // drawTicks: true
              },
              ticks: {
                callback: (value) => {
                  if (value.length > 8) {
                    return value.substr(0, 7) + "…";
                  } else {
                    return value;
                  }
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
  methods: {
    async getData() {
      // 月ごとにリクエスト
      const chartData = await axios
        .get("/mock/chart/topGraph.json")
        .then((response) => {
          // 降順にソートして返す
          return response.data.sort((a, b) => {
            return a.data < b.data ? 1 : -1;
          });
        })
        .catch((error) => console.log(error));

      this.datasets = [
        {
          label: "売上",
          data: chartData.map((item) => {
            return item.data;
          }),
          backgroundColor: "#0EBAB6",
          barPercentage: 0.3,
        },
      ];

      this.labels = chartData.map((item) => {
        return item.name;
      });
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
