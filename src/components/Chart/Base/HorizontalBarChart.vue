<script>
import { HorizontalBar, mixins } from "vue-chartjs";
const { reactiveProp } = mixins;

export default {
  extends: HorizontalBar,
  mixins: [reactiveProp],
  props: {
    responseData: {
      type: Array,
      default: () => []
    },
    chartData: {
      type: Object,
      default: () => {}
    },
    height: {
      type: Number,
      default: 220
    },
    options: {
      type: Object,
      default: function() {
        return {
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
                    value = `${this.responseData[index].order}：${value}`
                    return value.length > 6 ? value.substr(0, 5) + "…" : value
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
      }
    }
  },
  mounted () {
    this.renderChart(this.chartData, this.options);
  }
}
</script>
