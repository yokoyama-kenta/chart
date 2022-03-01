<template>
  <article class="chart-item">
    <p class="pl-1">上位グラフ</p>
    <HorizontalBarChart
      v-if="isLoaded"
      :chartData="{
        datasets,
        labels
      }"
      :responseData="responseData"
    />
  </article>
</template>

<script>
import axios from "axios";
import HorizontalBarChart from "./Base/HorizontalBarChart.vue";

export default {
  components: {
    HorizontalBarChart,
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
        .get("/mock/chart/topGraph.json")
        .then((response) => {
          return response.data;
        })
        .catch((error) => console.log(error));

      this.datasets = [
        {
          label: "売上",
          data: this.responseData.map((item) => {
            return item.profit;
          }),
          backgroundColor: "#0EBAB6",
          barPercentage: 0.3,
          maxBarThickness: 30,
        },
      ];

      this.labels = this.responseData.map((item) => {
        return item.branch_name;
      });
    },
  },
};
</script>

<style lang="scss" scoped></style>
