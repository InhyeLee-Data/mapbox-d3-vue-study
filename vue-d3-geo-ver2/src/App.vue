<template>
<div class='App'>
  <div class="header">
    <el-select v-model="colorVar">
      <el-option
        v-for="v in colorVars"
        :key="v"
        :label="v"
        :selected="v == colorVar"
        :value="v"
      ></el-option>
    </el-select>
    <div class="legend">
      <div v-for="color in colorDomain" :key ="color" class="legend-item">
        <div class="legend-item-color" :style="{backgroundColor: colorScale(color)}"></div>
        <span class="legend-item-text">{{color}}</span>
      </div>
    </div>
  </div>
  <el-divider></el-divider>

  <div class="maps">
    <el-card>
      <GeoJSON 
        :height="height" 
        :width="width" 
        :map="map"
        :colorVar="colorVar"
        :colorScale="colorScale"/>
    </el-card>
    <el-card>
      <MapboxMap
        :map="map"
        :colorVar="colorVar"
        :colorScale="colorScale"
        :colorDomain="colorDomain"/>
    </el-card>
  </div>
</div>
</template>

<script>
import * as d3 from "d3";
// import mapboxgl from "mapbox-gl";
import GeoJSON from './components/GeoJSON.vue';
import MapboxMap from './components/MapboxMap.vue';
import map from './africa.geo.json'
export default {
  name: 'App',
  components: {
    GeoJSON,
    MapboxMap
  },
  data() {
    return {
      map: { // geojson data;
        ...map, 
        features: map.features.map(d => ({ 
          ...d, 
          properties: {
            ...d.properties,
           centroid: d3.geoCentroid(d) // adding a new property
          }
        }))
      },
      // map: map, // This itself imports the entire json;
      colorVar: 'economy',
      colorVars: ['economy', 'income_grp', 'subregion'],
      width: 800,
      height: 1000,
      mb: null //it's going to be a mapgl instance later to be defined;
    }
  },
  mounted() {
    // (3) Let's see data in console
    console.log(this.map);
   },
  computed: {
    colorDomain() {
      return Array.from(new Set(map.features.map(d => d.properties[this.colorVar]))).sort();
    },
    colorScale() {
      return d3.scaleOrdinal(d3.schemeBrBG[this.colorDomain.length])
        .domain(this.colorDomain)
    },
  },
  methods: {

  }
}
</script>

<style>
  body {
    padding: 0;
    margin: 0;
    font-family: sans-serif;
  }
  .App {
    display: flex;
    flex-direction: column;
  }
  .header {
    padding: 10px;
    display: flex;
  }
  .legend {
    margin: 10px;
    display: flex;
  }
  .legend-item {
    margin-right: 10px; 
  }
  .legend-item-text {
    font-size: 0.8rem;
    font-family: "Open Sans", sans-serif;
    vertical-align: super; 
  }
  .legend-item-color {
    height: 20px;
    width: 20px;
    margin-right: 5px;
    display: inline-block;
  }
  .maps {
    display: flex;
    justify-content: space-around;
  }
  #mapbox-container {
    /* height: 500px;
    width: 400px; */
    height: 1000px;
    width: 800px;
  }
  text {
    transition: fill-opacity 0.5s;
    font-size: 12pt;
    text-anchor: middle;
    pointer-events: none;
  }
  path {
    fill-opacity: 0.8;
    stroke: black;
  }
  path:hover {
    fill-opacity: 1;
  }
</style>