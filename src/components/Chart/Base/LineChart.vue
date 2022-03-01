<script>
import { Line, mixins } from "vue-chartjs";
import { format } from "date-fns"
const { reactiveProp } = mixins;

export default {
  extends: Line,
  mixins: [reactiveProp],
  props: {
    responseData: {
      type: Object,
      default: () => {}
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
                      ? format(value, "yyyy/MM")
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
                  color: "#E6E6E6",
                  zeroLineColor: "#E6E6E6",
                  // drawOnChartArea: false,
                  // display: false,
                  // drawOnChartArea: false
                  drawTicks: false,
                  drawBorder: false,
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
                  stepSize: 40
                },
              },
            ],
          },
          legendCallback: function(chart) {
            const template = []
            chart.data.datasets.forEach((data, index) => {
              template.push(`
                <li>
                  <span class="border" style="background-color: ${data.borderColor}"></span>
                  <span>${index + 1}</span>
                </li>
              `)
            });
            return template.join("");
          },
          legend: {
            display: false,
            position: "right",
            align: "end",
          },
          tooltips: {
            mode: "label", //マウスオーバー時に表示されるtooltip
            callbacks: {
              title: (tooltipItem) => {
                return format(tooltipItem[0].xLabel, "yyyy年MM月");
              },
            },
          },
        };
      }
    },
    
  },
  mounted () {
    this.renderChart(this.chartData, this.options);

    const legend = this.generateLegend();
    this.$emit('sendLegend', legend);
  }
}
</script>
