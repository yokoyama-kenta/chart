<script>
import { Line, mixins } from "vue-chartjs";
import { format } from "date-fns";

const { reactiveProp } = mixins;

export default {
  extends: Line,
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
    currentChartIndex: {
      type: Number,
      default: 0
    },
    lastChartIndex: {
      type: Number,
      default: 1
    },
    options: {
      type: Object,
      default: function() {
        return {
          responsive: true,
          maintainAspectRatio: false,
          hover: {
            animationDuration: 50
          },
          scales: {
            xAxes: [
              {
                gridLines: {
                  display: false,
                },
                ticks: {
                  beginAtZero: true,
                  callback: (value, index, values) => {
                    return index === this.currentChartIndex ? format(value, 'MM/dd') : value === values[values.length - 1] ? '今日' : ''
                  }
                }
              },
            ],
            yAxes: [
              {
                gridLines: {
                  color: "transparent",
                  zeroLineColor: "#E6E6E6",
                  drawOnChartArea: false,
                  drawTicks: false
                },
                ticks: {
                  display: false,
                  callback: (value, index, values) => {
                    return value === values[values.length - 1] ? `${value}万円` : value.toLocaleString()
                  }
                }
              },
              {
                gridLines: {
                  color: "transparent",
                  zeroLineColor: "#E6E6E6",
                  drawTicks: false
                },
                ticks: {
                  display: false
                }
              },
            ]
          },
          legend: {
            display: false
          },
          tooltips:{
            mode: "label",
            enabled: false,
            intersect: false,
          },
          onHover: (evt, item) => {
            const init = () => {
              // 初回実行
              if (!this.border.canvas) {
                this.createCanvas()

                const chart = this.$data._chart
                chart.canvas.addEventListener('mouseleave', this.onLeave)

                this.onLeave()
              }
            }

            const draw = () => {
              this.onMove()
            }

            init()
            draw()

            if (item.length) {
              this.$emit("sync", item)
            }
          }
        }
      }
    },
  },
  data() {
    return {
      // ホバーした際の縦軸用
      border: {
        canvas: null,
        left: 0,
        lastActive: []
      }
    };
  },
  methods: {
    createCanvas() {
      const chart = this.$data._chart

      this.border.canvas = document.createElement("canvas");
      this.border.canvas.width = chart.canvas.offsetWidth
      this.border.canvas.height = chart.canvas.offsetHeight
      this.border.canvas.style.position = 'absolute'
      this.border.canvas.style.top = 0
      this.border.canvas.style.left = 0
      this.border.canvas.style.zIndex = -1
      this.border.canvas.style.pointerEvents = 'none'
      chart.canvas.parentNode.appendChild(this.border.canvas)
    },
    onLeave() {
      const chart = this.$data._chart
      this.border.lastActive.forEach(element => {
        element._model.radius = 0
      })
      this.border.left = 0

      const ctx = this.border.canvas.getContext("2d");
      ctx.clearRect(
        0,
        0,
        chart.canvas.offsetWidth,
        chart.canvas.offsetHeight
      )

      this.border.lastActive = []
    },
    onMove() {
      const chart = this.$data._chart
      const ctx = this.border.canvas.getContext("2d");
      const activeToolTip = chart.tooltip._active
      if (activeToolTip && activeToolTip.length) {

        // ぼやけ対策で小数点切り捨て後に+0.5px
        const left = Math.floor(activeToolTip[0].tooltipPosition().x) + 0.5;
        const top = chart.scales['y-axis-0'].top;
        const bottom = chart.scales['y-axis-0'].bottom;

        if (left !== this.border.left) {
          activeToolTip.forEach((element, index) => {
            const data = chart.data.datasets[index]
            element._model.radius = data.hoverRadius
          })
          this.border.lastActive.forEach((element) => {
            element._model.radius = 0
          })

          this.border.lastActive = activeToolTip

          ctx.clearRect(0, 0, chart.canvas.offsetWidth, chart.canvas.offsetHeight)
          ctx.save();
          ctx.beginPath();
          ctx.setLineDash([2, 2])
          ctx.moveTo(left, top);
          ctx.lineTo(left, bottom);
          ctx.lineWidth = 0.5;
          ctx.strokeStyle = '#8B99B2';
          ctx.stroke();
          ctx.restore();
        }
        this.border.left = left
      }
    }
  },
  beforeDestroy() {
    const chart = this.$data._chart;
    if (chart && chart.canvas) {
      chart.canvas.removeEventListener("mouseleave", this.onLeave);
    }
  },
  mounted () {
    this.renderChart(this.chartData, this.options);

    const legend = this.generateLegend();
    this.$emit('sendLegend', legend);
  }
}
</script>
