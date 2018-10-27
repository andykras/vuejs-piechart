<template>
  <svg class="pie"
       :viewBox="viewBox"
       @mousemove="moving">
    <circle v-for="(item,index) in dataObjects"
            :key="index"
            v-bind:style="{strokeDasharray: `${item.relativeSize} ${circleLength}`, strokeDashoffset: item.offset}"
            r="25%"
            cx="50%"
            cy="50%" />
    <text :x="radius"
          :y="-width*0.85">
      {{x && `x: ${x.toFixed(2)}`}}
    </text>
    <text :x="radius"
          :y="-width*0.7">
      {{y && `y: ${y.toFixed(2)}`}}
    </text>
  </svg>
</template>

<script>
export default {
  name: 'PieChart',
  mounted() {
    setInterval(_ => this.shuffle(this.data), 3000)
  },
  data() {
    return {
      data: [10, 5, 5, 5, 50, 5, 5, 5, 10],
      width: 100,
      x: null,
      y: null
    }
  },
  methods: {
    shuffle(data) {
      let dataCopy = data.slice()
      let temp
      let index
      let randomIndex

      for (index = 0; index < dataCopy.length; index++) {
        randomIndex = Math.floor(Math.random() * index)

        temp = dataCopy[index]
        dataCopy[index] = dataCopy[randomIndex]
        dataCopy[randomIndex] = temp
      }

      this.data = dataCopy
    },
    moving(event) {
      const svg = this.$el
      const pt = svg.createSVGPoint()
      ;[pt.x, pt.y] = [event.clientX, event.clientY]
      ;({ x: this.x, y: this.y } = pt.matrixTransform(svg.getScreenCTM().inverse()))
    }
  },
  computed: {
    dataTotal() {
      return this.data.reduce((previous, current) => previous + current, 0)
    },
    dataObjects() {
      let startingPoint = 0

      return this.data.map(item => {
        let relativeSize = (item / this.dataTotal) * this.circleLength

        let dataObject = { relativeSize: relativeSize, offset: -startingPoint }
        startingPoint += relativeSize

        return dataObject
      })
    },
    radius() {
      return this.width / 4
    },
    circleLength() {
      return 2 * Math.PI * this.radius
    },
    viewBox() {
      return `0 0 ${this.width} ${this.width}`
    }
  }
}
</script>

<style lang="scss">
$colors: red, yellow, cyan, green, blue, magenta, gray, purple, black;
svg {
  width: 350px;
  transform: rotate(-90deg);
  &.pie circle {
    fill: none;
    stroke-width: 50%; // to control center hole
    transition: stroke-dasharray 0.3s ease-in-out, stroke-dashoffset 0.3s ease-in-out;
  }
  &.pie text {
    transform: rotate(90deg);
    fill: white;
  }

  @for $i from 1 through length($colors) {
    &.pie circle:nth-child(#{$i}) {
      stroke: nth($colors, $i);
    }
  }
}
</style>
