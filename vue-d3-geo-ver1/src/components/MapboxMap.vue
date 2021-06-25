<template>
    <div id="mapbox-container"></div>
</template>
<script>
import * as d3 from "d3";

export default {
  name: 'MapboxMap',
  props: {
    height: Number,
    width: Number,
    map: Object,
    colorScale: Function,
    colorVar: String, // parameter for colorScale func
  },
}
</script>

<style scoped>

</style>