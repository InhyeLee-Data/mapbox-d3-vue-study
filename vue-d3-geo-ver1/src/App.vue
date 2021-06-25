<template>
<div class='App'>
  <div class="header">
    <el-select v-model="colorVar" @change="setColorVar">
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
      <!-- <MapboxMap/> -->
      <div id="mapbox-container"></div>
    </el-card>
  </div>
</div>
</template>

<script>
import * as d3 from "d3";
import mapboxgl from "mapbox-gl";
import GeoJSON from './components/GeoJSON.vue';
// import MapboxMap from './components/MapboxMap.vue';
import map from './africa.geo.json'
export default {
  name: 'App',
  components: {
    GeoJSON,
    // MapboxMap
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
    // (0) accessToken: https://dev.to/hmintoh/how-to-mapbox-with-vue-js-2a34
    mapboxgl.accessToken = process.env.VUE_APP_MAPBOX_TOKEN;

    // (1) this.mb is A basic New Map 
    this.mb = new mapboxgl.Map({
      container: "mapbox-container",
      style: "mapbox://styles/mapbox/streets-v11",
      center: [18, 4],
      zoom: 2,
      maxBounds: [
        [-18, -40],
        [53, 40],
      ],
    });
    
    // (2) Load Data as a Source and Add additional Layer to the Source
    this.mb.on('load', () => {
      // Get the layer information
      let layers = this.mb.getStyle().layers;
      // Find the index of the first symbol layer in the map style
      let firstSymbolId;
      for (let i = 0; i < layers.length; i++) {
        if (layers[i].type === 'symbol') {
          firstSymbolId = layers[i].id;
          break;
        }
      }
      // for (let i = 0; i < layers.length; i++) {
      //   console.log( layers[i]); // What's there in each layer
      // }
      this.mb.addSource("countryPaths", {
        type: "geojson",
        data: map
      });
      
      // addLayer takes two arguments (1) Layer as an Object (2)
      this.mb.addLayer({
        id: 'countries',
        type: 'fill',
        source: 'countryPaths',
        layout: {},
        paint: {
          "fill-opacity": 0.5,
          "fill-color": this.mapboxFillColorSpec
        }
      }, firstSymbolId) // Add Layer here as the argument (https://docs.mapbox.com/mapbox-gl-js/example/geojson-layer-in-stack/)
    })

    // (3) Let's see data in console
    console.log(this.map)
   },
  computed: {
    colorDomain() {
      return Array.from(new Set(map.features.map(d => d.properties[this.colorVar]))).sort();
    },
    colorScale() {
      return d3.scaleOrdinal(d3.schemeBrBG[this.colorDomain.length])
        .domain(this.colorDomain)
    },
    mapboxFillColorSpec() {
      return [
        "to-color", [
          "at", 
          [
            "index-of", 
            [
              "get", 
              this.colorVar
            ],
            [
              "literal",
              this.colorDomain
            ]
          ], 
          [
            "literal", 
            this.colorScale.range()
          ]
        ]
      ]
    }
  },
  methods: {
    setColorVar() {
      this.mb.setPaintProperty('countries', 'fill-color', this.mapboxFillColorSpec) 
      // ID of the layer, name (paint property), value
    }
  }
}
</script>

<style>
  body {
    padding: 0;
    margin: 0;
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