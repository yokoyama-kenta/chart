<template>
  <article class="chart-item">
    <p class="pl-1">処方箋枚数</p>
    <div class="chart-wrapper">
      <!-- <div>
        <p>{{display.current.date}} (今週): {{display.current.count}}枚</p>
        <p>{{display.last.date}} (先週): {{display.last.count}}枚</p>
      </div> -->
      <LineSyncChart
        class="line-chart"
        v-if="isLoaded"
        :chartData="{
          datasets,
          labels
        }"
        :responseData="responseData"
        :currentChartIndex="currentChartIndex"
        :lastChartIndex="lastChartIndex"
        @sync="sync"
      />
    </div>
  </article>
</template>

<script>
  import axios from 'axios'
  import LineSyncChart from './Base/LineSyncChart.vue'
  import { addDays, parse, format } from "date-fns"

  export default {
    components: {
      LineSyncChart
    },
    data () {
      return {
        responseData: null,
        datasets: [],
        labels: [],
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
        isLoaded: false,
        // 現在グラフが0番目、過去グラフが1番目と決めうち
        currentChartIndex: 0,
        lastChartIndex: 1
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
        this.responseData = await axios.get('/mock/chart/dashBoard.json')
          .then(response => {
            return response.data
          })
          .catch(error => console.log(error))

        this.datasets = this.responseData.map((item, index) => {
          return {
            label: item.name,
            borderColor: index === this.currentChartIndex ? '#0EBAB6' : '#A4ACBA',
            pointBackgroundColor: index === this.currentChartIndex ? '#0EBAB6' : '#A4ACBA',
            backgroundColor: "transparent",
            data: item.count,
            lineTension: 0,
            borderWidth: 1.5,
            pointRadius: 0,
            pointStyle: '',
            pointHitRadius: 0,
            hoverRadius: 2, // 丸ポチのサイズ
          }
        })

        this.labels = this.responseData[this.currentChartIndex].count.map((item, index) => {
          return addDays(parse(this.responseData[this.currentChartIndex].start, "yyyy-MM-dd", new Date()), index)
        })
      },
      // ホバー時に同期したい処理
      sync(chart) {
        const activeToolTip = chart.tooltip._active
        if (activeToolTip && activeToolTip[this.currentChartIndex]) {
          // カウント数反映
          const index = activeToolTip[this.currentChartIndex]._index
          this.display.current.count = this.responseData[this.currentChartIndex].count[index]
          this.display.last.count = this.responseData[this.lastChartIndex].count[index]

          // 日付反映
          this.display.current.date = format(addDays(parse(this.responseData[this.currentChartIndex].start, "yyyy-MM-dd", new Date()), index), "yyyy-MM-dd")
          this.display.last.date = format(addDays(parse(this.responseData[this.lastChartIndex].start, "yyyy-MM-dd", new Date()), index), "yyyy-MM-dd")
        }
      }
    }
  }
</script>

<style scoped>
.chart-wrapper {
  display: flex;
  flex-direction: column;
}
.line-chart {
  flex-grow: 1;
  /* height: 100%; */
  position: relative;
  z-index: 0;
  background: #fff;
}
</style>
