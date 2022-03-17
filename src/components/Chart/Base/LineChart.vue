<script>
import { Line, mixins } from "vue-chartjs";
import { format } from "date-fns";
const { reactiveProp } = mixins;
import "chartjs-plugin-style";

export default {
  extends: Line,
  mixins: [reactiveProp],
  props: {
    responseData: {
      type: Object,
      default: () => {},
    },
    chartData: {
      type: Object,
      default: () => {},
    },
    height: {
      type: Number,
      default: 220,
    }
  },
  data() {
    return {
      // ホバーした際の縦軸用
      border: {
        canvas: null,
        left: 0,
        lastActive: [],
      },
      currentIndex: null
    };
  },
  computed: {
    options() {
      return {
        responsive: true,
        maintainAspectRatio: false,
        hover: {
          animationDuration: 50,
        },
        scales: {
          xAxes: [
            {
              gridLines: {
                color: "transparent",
                zeroLineColor: "transparent",
                drawTicks: false,
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
                fontColor: "#848484",
              },
            },
          ],
          yAxes: [
            {
              gridLines: {
                color: "#E6E6E6",
                zeroLineColor: "#E6E6E6",
                drawTicks: false,
                drawBorder: false,
              },
              ticks: {
                beginAtZero: true,
                callback: (value, index, values) => {
                  return value === values[values.length - 1]
                    ? `${value}万円`
                    : value.toLocaleString();
                },
                padding: 30,
                stepSize: 40,
              },
            },
          ],
        },
        legendCallback: (chart) => {
          const template = [];
          const total = chart.data.datasets.map(element => {
            return element.data.reduce((sum, data) => {
              return sum + data;
            }, 0)
          });

          template.push(`
            <div class="label">
              <span>${this.currentIndex === null
                ? format(chart.data.labels[0], "yyyy/MM") + '~' + format(chart.data.labels[chart.data.labels.length - 1], "yyyy/MM")
                : format(chart.data.labels[this.currentIndex], "yyyy/MM")}
              </span>
              <ul>
          `)
          chart.data.datasets.forEach((data, index) => {
            template.push(`
              <li>
                <span class="border" style="background-color: ${
                  data.borderColor
                }"></span>
                <span>${index + 1}: </span>
                <span>${this.currentIndex === null ? total[index] : data.data[this.currentIndex]}</span>
              </li>
            `);
          });
          template.push(`
              </ul>
            </div>
          `)
          return template.join("");
        },
        legend: {
          display: false,
          position: "right",
          align: "end",
        },
        tooltips: {
          mode: "label", //マウスオーバー時に表示されるtooltip
          intersect: false,
          enabled: false,
          caretSize: 0,
          backgroundColor: "#FFF",
          bodyFontColor: "#504946",
          titleFontColor: "#504946",
          borderColor: "#D8D8D8",
          borderWidth: 1,
          shadowOffsetX: 1,
          shadowOffsetY: 1,
          shadowBlur: 3,
          shadowColor: "#15223226",
          callbacks: {
            title: (tooltipItem) => {
              return format(tooltipItem[0].xLabel, "yyyy年MM月");
            },
          },
        },
        onHover: () => {
          const init = () => {
            // 初回実行
            if (!this.border.canvas) {
              this.createCanvas();

              const chart = this.$data._chart;
              chart.canvas.addEventListener("mouseleave", this.onLeave);

              this.onLeave();
            }
          };

          const draw = () => {
            this.onMove();
          };

          init();
          draw();
        },
      };
    }
  },
  methods: {
    createCanvas() {
      const chart = this.$data._chart;

      this.border.canvas = document.createElement("canvas");
      this.border.canvas.width = chart.canvas.offsetWidth;
      this.border.canvas.height = chart.canvas.offsetHeight;
      this.border.canvas.style.position = "absolute";
      this.border.canvas.style.top = 0;
      this.border.canvas.style.left = 0;
      this.border.canvas.style.zIndex = -1;
      this.border.canvas.style.pointerEvents = "none";
      chart.canvas.parentNode.appendChild(this.border.canvas);
    },
    onLeave() {
      const chart = this.$data._chart;
      this.border.lastActive.forEach((element) => {
        element._model.radius = 0;
      });
      this.border.left = 0;

      const ctx = this.border.canvas.getContext("2d");
      ctx.clearRect(0, 0, chart.canvas.offsetWidth, chart.canvas.offsetHeight);

      this.border.lastActive = [];
      this.currentIndex = null
    },
    onMove() {
      const chart = this.$data._chart;
      const ctx = this.border.canvas.getContext("2d");
      const activeToolTip = chart.tooltip._active;
      if (activeToolTip && activeToolTip.length) {
        // ぼやけ対策で小数点切り捨て後に+0.5px
        const left = Math.floor(activeToolTip[0].tooltipPosition().x) + 0.5;
        const top = chart.scales["y-axis-0"].top;
        const bottom = chart.scales["y-axis-0"].bottom;

        if (left !== this.border.left) {
          activeToolTip.forEach((element, index) => {
            const data = chart.data.datasets[index];
            element._model.radius = data.hoverRadius;
          });
          this.border.lastActive.forEach((element) => {
            element._model.radius = 0;
          });

          this.border.lastActive = activeToolTip;

          ctx.clearRect(
            0,
            0,
            chart.canvas.offsetWidth,
            chart.canvas.offsetHeight
          );
          ctx.save();
          ctx.beginPath();
          ctx.setLineDash([2, 2]);
          ctx.moveTo(left, top);
          ctx.lineTo(left, bottom);
          ctx.lineWidth = 0.5;
          ctx.strokeStyle = "#8B99B2";
          ctx.stroke();
          ctx.restore();

          this.currentIndex = activeToolTip[0]._index
        }
        this.border.left = left;
      }
    },
  },
  beforeDestroy() {
    const chart = this.$data._chart;
    if (chart && chart.canvas) {
      chart.canvas.removeEventListener("mouseleave", this.onLeave);
    }
  },
  mounted() {
    this.renderChart(this.chartData, this.options);

    const legend = this.generateLegend();
    this.$emit("sendLegend", legend);
  },
  watch: {
    currentIndex() {
      const legend = this.generateLegend();
      this.$emit("sendLegend", legend);
    }
  }
};
</script>
