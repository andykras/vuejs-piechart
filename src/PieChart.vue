<template>
  <svg class="pie"
       :viewBox="viewBox"
       @mousemove="moving">
    <circle v-for="(item,index) in dataObjects"
            :key="index"
            :style="{strokeDasharray: `${item.relativeSize} ${circleLength}`, strokeDashoffset: item.offset}"
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
    const fn = _ => !this.stabilization() && setTimeout(fn, 1000)
    fn()
  },
  data() {
    const L = Math.random() * 20
    for (var data = [], i = 0; i < L; ++i) data[i] = L * (1 + 9 * Math.random())
    return {
      data: data,
      width: 100,
      x: null,
      y: null
    }
  },
  methods: {
    stabilization() {
      this.data = this.data.map((_, index) => this.data[Math.floor(Math.random() * index)])
      return this.data.every((val, i, arr) => val === arr[0])
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
        const relativeSize = (item / this.dataTotal) * this.circleLength
        const dataObject = { relativeSize: relativeSize, offset: -startingPoint }
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
$colors: red, maroon, yellow, olive, lime, green, aqua, teal, blue, navy, fuchsia, purple;
$repeat: 20;
$dist: 3%;

*,
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}

svg {
  max-height: 100vh;
  max-width: 100vw;
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

  @for $i from 1 through $repeat {
    &.pie circle:nth-child(#{length($colors)}n + #{$i}) {
      @if $i % 2==0 {
        stroke: lighten(nth($colors, random(length($colors))), $dist);
      } @else {
        stroke: darken(nth($colors, random(length($colors))), $dist);
      }
    }
  }
}
</style>
