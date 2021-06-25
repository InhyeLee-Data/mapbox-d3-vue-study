<template>
  <svg :height="height" :width="width">
    <g class="paths" @mouseleave="onHover(null)">
      <path
        v-for="feature in map.features"
        :key="feature.properties.geounit"
        :d="geoPath(feature)"
        :fill="colorScale(feature.properties[colorVar])"
        @mouseenter="onHover(feature.properties)"
      ></path>
    </g>
    <text id="hover" :x="50" :y="50"></text>
  </svg>
</template>

<script>
import * as d3 from "d3";

export default {
  name: 'GeoJSON',
  props: {
    height: Number,
    width: Number,
    map: Object,
    colorScale: Function,
    colorVar: String, // parameter for colorScale func
  },
  computed: {
    projection() {
      return d3.geoEqualEarth()
        .center([18, 4])
        .scale(600)
        .translate([this.width/2, this.height/2]);
    },
    geoPath() {
      return d3.geoPath(this.projection);
    },
  },
  methods: {
    onHover(nextHover) {
      //nextHover = features.properties
      if (nextHover === null) {
        d3.select('#hover').text('')
        return;
      }
      const projectedCentroid = this.projection(nextHover.centroid)
     // console.log("projectedCentroid", projectedCentroid);
      d3.select('#hover').text(nextHover.name) // nextHover.admin
        .attr('x', projectedCentroid[0])
        .attr('y', projectedCentroid[1])
        .attr("font-family", "Arial")
        .attr("font-size", "small")
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.labeled-point text {
  opacity: 0;
  pointer-events: none;
}
.labeled-point:hover text {
  opacity: 1;
}
circle {
  stroke: black;
  stroke-width: 1px;
  opacity: 0.5;
}
</style>
