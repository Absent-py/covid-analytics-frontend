<template>
  <bar_chart :chartData="testData"></bar_chart>
</template>

<script>
import bar_chart from '/src/components/bar_chart.vue'

export default {
  name: 'Home',
  components: {
    bar_chart,
  },
  data: () => ({
    testData: {
      labels: ['Paris', 'Nîmes', 'Toulon', 'Perpignan', 'Autre'],
      datasets: [
        {
          data: [30, 40, 60, 70, 5],
          backgroundColor: ['#77CEFF', '#0079AF', '#123E6B', '#97B0C4', '#A5C8ED'],
        }
      ]
    }
  })
}
</script>
