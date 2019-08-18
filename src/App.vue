<template>
  <div id="app">
    <DynamicBarChart :barGapSize="10"
                     :data="data"
                     :iterationTimeout="100"
                     :startRunningTimeout="2500" />
  </div>
</template>

<script>
import DynamicBarChart from './components/DynamicBarChart.vue'
import helpers from './components/helpers'
import mocks from './mock/mocks'
export default {
  name: 'app',
  components: {
    DynamicBarChart
  },
  data () {
    return {
      data:
        helpers.generateData(100, mocks.defaultChart, {
          prefix: 'Iteration'
        })
    }
  }
}
</script>

<style lang="scss">
</style>
