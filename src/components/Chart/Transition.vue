<template>
  <div class="card">
    推移
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

  export default {
    components: {
      LineChart
    },
    data () {
      return {
        datasets: [],
        labels: [],
        isLoaded: false
      }
    },
    async mounted () {
      await this.getData()
      this.isLoaded = true
    },
    computed: {
      options() {
        return {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            xAxes: [
              {
                gridLines: {
                  color: "transparent",
                  zeroLineColor: "#E6E6E6",

                  // display: false,
                  drawTicks: false
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
                    return index === 0 || value === values[values.length - 1] ? moment(value).format('YYYY/MM') : ''
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

                  // display: false,
                  // drawOnChartArea: false
                  drawTicks: false
                  // display: false,
                  // drawTicks: true
                },
                ticks: {
                  beginAtZero: true,
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
            mode:'label' //マウスオーバー時に表示されるtooltip
          }
        }
      }
    },
    methods: {
      async getData () {
        // 月ごとにリクエスト
        const chartData = await axios.get('/mock/chart/transition.json')
          .then(response => {
            // 合計を降順にソートして返す
            response.data.data.sort(
              (a,b) => {
                const totalA = a.data.reduce((sum, element) => {
                  return sum + element
                }, 0)
                const totalB = b.data.reduce((sum, element) => {
                  return sum + element
                }, 0)
                return (totalA < totalB ? 1 : -1)
              }
            )
            return response.data
          })
          .catch(error => console.log(error))

        this.datasets = chartData.data.map(item => {
          return {
            label: item.name,
            borderColor: item.color,
            backgroundColor:"rgba(255,0,0,0)",
            data: item.data,
            lineTension: 0,
            borderWidth: 1.5,
            pointRadius: 0,
            pointHitRadius: 10
          }
        })

        this.labels = chartData.data[0].data.map((item, index) => {
          return moment(chartData.start).set('month', index).format('YYYY-MM-DD')
        })
      }
    }
  }
</script>

<style>
.card {
  width: 800px;
  height: 400px;
}
</style>
