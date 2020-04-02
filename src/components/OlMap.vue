<template>
  <div>
    <div id="map" class="map" />
    <button @click="addRoadMap">roadMapOn</button>
  </div>
</template>

<script>
import 'ol/ol.css'
import { Map, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
// import BingMaps from 'ol/source/BingMaps'
// import Stamen from 'ol/source/Stamen'
import XYZ from 'ol/source/XYZ'
// import WMS from 'ol/source/TileWMS'
export default {
  name: 'OlMap',
  data() {
    return {
      map: null,
      roadMap: false
    }
  },
  mounted() {
    this.initMap()
  },
  methods: {
    initMap() {
      const view = new View({ // 地图视图
        center: [104, 30], // 坐标
        zoom: 10,
        projection: 'EPSG:4326'
      })
      this.map = new Map({
        target: 'map', // 对应页面里id为map的元素
        layers: [// 图层
          new TileLayer({
            source: new OSM()
          })
        ],
        view: view
      })
    },
    addRoadMap() {
      var tiandituLuWang = new TileLayer({
        source: new XYZ({
          url: 'http://t3.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
        }),
        zIndex: 200
      })
      this.roadMap = !this.roadMap
      this.map.addLayer(tiandituLuWang) // 将给定的图层添加到地图顶部
    }
  }
}
</script>

<style>
  .map{
    height:800px;
    width:100%;
  }
</style>

