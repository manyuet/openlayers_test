<template>
  <div id="map" />
</template>

<script>
import 'ol/ol.css'
import { Map, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
// import OSM from 'ol/source/OSM'
import XYZ from 'ol/source/XYZ'
export default {
  name: 'Test',
  data() {
    return {
      map: null
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      this.map = new Map({
        target: 'map',
        layers: [
          new TileLayer({
            source: new XYZ({
              url: 'http://t4.tianditu.com/DataServer?T=vec_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6' // 图层数据源
            }),
            zIndex: 200,
            visible: true
          })
        ],
        view: new View({
          projection: 'EPSG:4326',
          center: [114.064839, 22.548857],
          zoom: 12
        })
      })
    }
  }
}
</script>

<style scoped>

</style>
