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
                const legends = this.chartData.datasets.map((data) => {
                  return {
                    text: this.responseData.data.find((item) => item.name === data.label)
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
  }
}
</script>
