<template>
  <div class="live-chart">
    <h1 v-if="showTitle">{{currentItem.name}}</h1>
    <section class="chart">
      <div class="chart-bars"
           :style="{ 'height': (barHeight + barGapSize) * Object.keys(currentValues).length +'px'}">
        <div class="bar-wrapper"
             v-for="(key, idx) in sortedCurrentValues"
             :style="{ 'transform': translateY((barHeight + barGapSize) * idx), 'transitionDuration': 200 / 1000+'s' }"
             :key='`bar_${key}`'>
          <label>
            {{
            !currentValues[key].label
            ? key
            : currentValues[key].label
            }}
          </label>
          <div class="bar"
               :style="{ 'height': barHeight+'px', 'width': widthStr(currentValues[key].value), 'background': typeof currentValues[key].color === 'string' ? currentValues[key].color : `linear-gradient(to right, ${currentValues[key].color.join(',')})` }" />
          <span class="value"
                :style="{ 'color': typeof currentValues[key].color === 'string' ? currentValues[key].color : currentValues[key].color[0] }">{{currentValues[key].value}}</span>
        </div>
      </div>
    </section>
  </div>

</template>

<script>
import { clearTimeout } from 'timers'
const getRandomColor = () => {
  const letters = '0123456789ABCDEF'
  let color = '#'
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)]
  }
  return color
}

export default {
  name: 'DynamicBarChart',
  props: {
    showTitle: {
      type: Boolean,
      default: true
    },
    iterationTimeout: {
      type: Number,
      default: 200
    },
    data: {
      type: Array,
      default () {
        return []
      }
    },
    startRunningTimeout: {
      type: Number,
      default: 0
    },
    barHeight: {
      type: Number,
      default: 50
    },
    barGapSize: {
      type: Number,
      default: 20
    },
    baseline: {
      type: Object
    }
  },
  data () {
    return {
      dataQueue: [],
      activeItemIdx: 0,
      highestValue: 0,
      currentValues: {},
      firstRun: false,
      iterationTimeoutHolder: null
    }
  },
  computed: {
    currentItem () {
      return this.dataQueue[this.activeItemIdx - 1] || {}
    },
    sortedCurrentValues () {
      const { currentValues } = this
      const keys = Object.keys(currentValues)
      return keys.sort((a, b) => currentValues[b].value - currentValues[a].value)
    }
  },
  watch: {
    dataQueue () {
      this.start()
    },
    activeItemIdx () {
      this.iterationTimeoutHolder = setTimeout(this.nextStep, 1000)
    }
  },
  methods: {
    widthStr (value) {
      const maxValue = this.highestValue / 0.85
      let width = Math.abs((value / maxValue * 100))
      let widthStr
      if (isNaN(width) || !width) {
        widthStr = '1px'
      } else {
        widthStr = `${width}%`
      }
      return widthStr
    },
    translateY (value) {
      return `translateY(${value}px)`
    },
    start () {
      if (this.activeItemIdx > 1) {
        return
      }
      this.nextStep(true)
    },
    setNextValues () {
      const { dataQueue, activeItemIdx, currentValues } = this
      if (!dataQueue[activeItemIdx]) {
        this.iterationTimeoutHolder = null
        return
      }

      const roundData = dataQueue[activeItemIdx].values
      const nextValues = {}
      let highestValue = 0
      roundData.map((c) => {
        nextValues[c.id] = {
          ...c,
          color: c.color || (currentValues[c.id] || {}).color || getRandomColor()
        }

        if (Math.abs(c.value) > highestValue) {
          highestValue = Math.abs(c.value)
        }

        return c
      })
      console.table(highestValue)
      this.currentValues = nextValues
      this.highestValue = highestValue
      this.activeItemIdx++
    },
    nextStep (firstRun = false) {
      this.firstRun = firstRun
      this.setNextValues()
    }
  },
  mounted () {
    this.dataQueue = this.data
  },
  beforeDestroy () {
    clearTimeout(this.iterationTimeoutHolder)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.live-chart {
  width: 100%;
  padding: 20px;
  box-sizing: border-box;
  position: relative;
  text-align: center;
  h1 {
    font-weight: 700;
    font-size: 60px;
    text-transform: uppercase;
    text-align: center;
    padding: 20px 10px;
    margin: 0;
  }

  .chart {
    position: relative;
    margin: 20px auto;
  }

  .chart-bars {
    position: relative;
    width: 100%;
  }

  .bar-wrapper {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    position: absolute;
    top: 0;
    left: 0;
    transform: translateY(0);
    transition: transform 0.5s linear;
    padding-left: 200px;
    box-sizing: border-box;
    width: 100%;
    justify-content: flex-start;

    label {
      position: absolute;
      height: 100%;
      width: 200px;
      left: 0;
      padding: 0 10px;
      box-sizing: border-box;
      text-align: right;
      top: 50%;
      transform: translateY(-50%);
      font-size: 16px;
      font-weight: 700;
      display: flex;
      justify-content: flex-end;
      align-items: center;
    }

    .value {
      font-size: 16px;
      font-weight: 700;
      margin-left: 10px;
    }

    .bar {
      width: 0%;
      transition: width 0.5s linear;
    }
  }
}
</style>
