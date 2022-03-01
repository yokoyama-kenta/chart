<template>
  <div class="card">
    処方箋枚数
    <p>{{display.current.date}} (今週): {{display.current.count}}枚</p>
    <p>{{display.last.date}} (先週): {{display.last.count}}枚</p>
    <LineChart
      v-if="isLoaded"
      :chartData="{
        datasets,
        labels
      }"
      :options="options"
    />
  </div>
</template>

<script>
  import axios from 'axios'
  import LineChart from './Base/LineChart.vue'
  import moment from 'moment'

  // 現在グラフが0番目、過去グラフが1番目と決めうち
  const currentChartIndex = 0
  const lastChartIndex = 1

  export default {
    components: {
      LineChart
    },
    data () {
      return {
        datasets: [],
        labels: [],
        options: {},
        display: {
          current: {
            date: null,
            count: 0
          },
          last: {
            date: null,
            count: 0
          },
        },
        isLoaded: false
      }
    },
    async mounted () {
      await this.getData()
      this.isLoaded = true
    },
    computed: {},
    methods: {
      async getData () {
        // 月ごとにリクエスト
        const chartData = await axios.get('/mock/chart/dashBoard.json')
          .then(response => {
            return response.data
          })
          .catch(error => console.log(error))

        this.datasets = chartData.map((item, index) => {
          return {
            label: item.name,
            borderColor: index === currentChartIndex ? '#0EBAB6' : '#A4ACBA',
            backgroundColor:"rgba(255,0,0,0)",
            data: item.count,
            lineTension: 0,
            borderWidth: 1.5,
            pointRadius: 0,
            pointStyle: '',
            pointHitRadius: 100,
            hoverRadius: 1.5,
            hoverBorderWidth: 2
          }
        })

        this.labels = chartData[currentChartIndex].count.map((item, index) => {
          return moment(chartData[currentChartIndex].start).add(index, 'days').format('YYYY-MM-DD')
        })

        this.options = {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            xAxes: [
              {
                gridLines: {
                  display: false,
                },
                ticks: {
                  beginAtZero: true,
                  callback: (value, index, values) => {
                    return index === currentChartIndex ? moment(value).format('MM/DD') : value === values[values.length - 1] ? '今日' : ''
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
            enabled: false
          },
          hover: {
            onHover: (evt, item) => {
              if (item.length) {
                // カウント数反映
                const index = item[currentChartIndex]._index
                this.display.current.count = chartData[currentChartIndex].count[index]
                this.display.last.count = chartData[lastChartIndex].count[index]

                // 日付反映
                this.display.current.date = moment(chartData[currentChartIndex].start).add(index, 'days').format('YYYY-MM-DD')
                this.display.last.date = moment(chartData[lastChartIndex].start).add(index, 'days').format('YYYY-MM-DD')

              }
            }
          },
        }
      }
    }
  }
</script>

<style>
.card {
  width: 600px;
  height: 400px;
}
</style>
