<template>
    <div id="mapbox-container"></div>
</template>
<script>
import * as d3 from "d3";
import mapboxgl from "mapbox-gl";

export default {
  name: 'MapboxMap',
  data() {
      return {
          mb: null,

      }
  },
  props: {
    height: Number,
    width: Number,
    map: Object,
    colorScale: Function,
    colorVar: String, // parameter for colorScale func
    colorDomain: Array,
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
        data: this.map
      });
      
      // addLayer takes two arguments (1) Layer as an Object (2) Layer
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
  },
  computed: {
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
  },
  watch: {
      // Now, Watch the "colorVar" coming from the Parent.
      // Whenever the colorVar changes in the Parent, call setColorVar above
      "colorVar": function() {
          this.setColorVar();
      }
  }
}
</script>

<style scoped>

</style>