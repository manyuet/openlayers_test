<template>
  <div>
    <div id="map" class="map" />
    <button @click="addLayer">addLayer</button>
    <button @click="addLayer1">addLayers</button>
  </div>
</template>

<script>
import 'ol/ol.css'
import { Map, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
// import OSM from 'ol/source/OSM'
import XYZ from 'ol/source/XYZ'
import { fromLonLat } from 'ol/proj.js'
export default {
  name: 'OlMap',
  data() {
    return {
      map: null
    }
  },
  mounted() {
    this.initMap()
  },
  methods: {
    initMap() {
      const view = new View({ // 地图视图
        center: fromLonLat([116.62, 23.67]), // 坐标
        zoom: 1
      })
      this.map = new Map({
        target: 'map', // 对应页面里id为map的元素
        layers: [// 图层
          new TileLayer({
            source: new XYZ({
              url: 'http://t4.tianditu.com/DataServer?T=vec_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6' // 图层数据源
            }),
            zIndex: 200,
            visible: true
          })
        ],
        view: view
      })
    },
    addLayer() {
      var test = new TileLayer({
        source: new XYZ({
          url: 'http://t3.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
        }),
        zIndex: 200
      })
      this.map.addLayer(test)
    },
    addLayer1() {
      var test1 = new TileLayer({
        source: new XYZ({
          url: 'http://t3.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
        }),
        zIndex: 200
      })
      this.map.addLayer(test1)
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

