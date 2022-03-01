<template>
  <div class="card">
    推移
    <LineChart
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
import LineChart from "./Base/LineChart.vue";
import moment from "moment";

export default {
  components: {
    LineChart,
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
        .get("/mock/chart/transition.json")
        .then((response) => {
          return response.data;
        })
        .catch((error) => console.log(error));

      this.datasets = chartData.data.map((item) => {
        return {
          label: item.name,
          borderColor: item.color,
          backgroundColor: "rgba(255,0,0,0)",
          data: item.data,
          lineTension: 0,
          borderWidth: 1.5,
          pointRadius: 0,
          pointHitRadius: 10,
          hoverRadius: 1.5,
        };
      });

      this.labels = chartData.data[0].data.map((item, index) => {
        return moment(chartData.start).set("month", index).format("YYYY-MM-DD");
      });

      this.options = {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          xAxes: [
            {
              gridLines: {
                color: "transparent",
                zeroLineColor: "transparent",
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
                callback: (value, index, values) => {
                  return index === 0 || value === values[values.length - 1]
                    ? moment(value).format("YYYY/MM")
                    : "";
                },
                padding: 15,
                maxRotation: 0,
                minRotation: 0,
              },
            },
          ],
          yAxes: [
            {
              gridLines: {
                color: "transparent",
                zeroLineColor: "#E6E6E6",
                drawOnChartArea: false,
                // display: false,
                // drawOnChartArea: false
                drawTicks: false,
                // display: false,
                // drawTicks: true
              },
              ticks: {
                beginAtZero: true,
                callback: (value, index, values) => {
                  return value === values[values.length - 1]
                    ? `${value}万円`
                    : value.toLocaleString();
                },
                padding: 30,
              },
            },
            {
              gridLines: {
                // color: "transparent",
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
          display: true,
          position: "right",
          labels: {
            padding: 35,
            boxWidth: 30,
            generateLabels: () => {
              const legends = this.datasets.map((data) => {
                return {
                  text: chartData.data.find((item) => item.name === data.label)
                    .order,
                  // 凡例ボックスの塗りつぶしスタイル
                  fillStyle: data.borderColor,
                  //  trueの場合、この項目は非表示のデータセットを表します。ラベルは取り消し線を伴ってレンダリングされます
                  hidden: false,
                  // ボックス枠用。次をご覧ください。https://developer.mozilla.org/en/docs/Web/API/CanvasRenderingContext2D/lineCap
                  lineCap: "",
                  // ボックス枠用。次をご覧ください。https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/setLineDash
                  lineDash: [0],
                  // ボックス枠用。次をご覧ください。https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/lineDashOffset
                  lineDashOffset: 0,
                  // ボックス枠用。次をご覧ください。https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/lineJoin
                  lineJoin: "bevel",
                  // 枠線の幅
                  lineWidth: 0,
                  // 凡例ボックスのストロークスタイル
                  strokeStyle: "transparent",
                  // 凡例ボックスのポイントスタイル（usePointStyleがtrueの場合にのみ使用されます）
                  pointStyle: "",
                };
              });

              return legends;
            },
          },
        },
        tooltips: {
          mode: "label", //マウスオーバー時に表示されるtooltip
          callbacks: {
            title: (tooltipItem) => {
              return moment(tooltipItem[0].label).format("YYYY年MM月DD日");
            },
          },
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
