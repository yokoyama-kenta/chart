<template>
  <article class="chart-item">
    <p class="pl-1">推移</p>
    <LineChart
      v-if="isLoaded"
      :chartData="{
        datasets,
        labels,
      }"
      :responseData="responseData"
    />
  </article>
</template>

<script>
import axios from "axios";
import LineChart from "./Base/LineChart.vue";
import { addMonths, parse } from "date-fns"

export default {
  components: {
    LineChart,
  },
  data() {
    return {
      responseData: null,
      datasets: [],
      labels: [],
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
      this.responseData = await axios
        .get("/mock/chart/transition.json")
        .then((response) => {
          return response.data;
        })
        .catch((error) => console.log(error));

      this.datasets = this.responseData.data.map((item) => {
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

      this.labels = this.responseData.data[0].data.map((item, index) => {
        return addMonths(parse(this.responseData.start, "yyyy-MM-dd", new Date()), index)
      });
    },
  },
};
</script>

<style lang="scss" scoped></style>
