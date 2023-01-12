<template>
  <BarChart :chartData="data" />
</template>

<script>
import { BarChart } from 'vue-chart-3'
import { Chart, registerables } from "chart.js"

Chart.register(...registerables)
export default {
  name: "bar_chart",
  components: {
    BarChart
  },
  props: {
    data: Object
  }
}
</script>

