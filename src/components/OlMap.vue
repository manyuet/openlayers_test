<template>
  <div>
    <div id="map" class="map" />
    <!--    <button @click="moveToLeft">Left</button>-->
    <!--    <button @click="addLayer">addLayer</button>-->
    <!--    <button @click="addLayer1">addLayers</button>-->
    <!--    <button @click="display">display</button>-->
    <input type="radio" checked="checked" name="mapSource" @click="switchToOSM">OpenStreetMap地图
    <input type="radio" name="mapSource" @click="switchToBing">Bing地图
    <input type="radio" name="mapSource" @click="switchToStamen">Stamen地图
  </div>
</template>

<script>
import 'ol/ol.css'
import { Map, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
import BingMaps from 'ol/source/BingMaps'
import Stamen from 'ol/source/Stamen'
// import XYZ from 'ol/source/XYZ'
// import { fromLonLat } from 'ol/proj.js'
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
        center: [104.06, 30.67], // 坐标
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
    switchToOSM() {
      var openStreetMapLayer = new TileLayer({
        source: new OSM()
      })
      this.map.removeLayer(this.map.getLayers().item(0))
      this.map.addLayer(openStreetMapLayer)
    },
    switchToBing() {
      var bingMapLayer = new TileLayer({
        source: new BingMaps({
          key: 'AkjzA7OhS4MIBjutL21bkAop7dc41HSE0CNTR5c6HJy8JKc7U9U9RveWJrylD3XJ',
          imagerySet: 'Road'
        })
      })
      this.map.removeLayer(this.map.getLayers().item(0))
      this.map.addLayer(bingMapLayer)
    },
    switchToStamen() {
      var stamentLayer = new TileLayer({
        source: new Stamen({
          layer: 'watercolor'
        })
      })
      this.map.removeLayer(this.map.getLayers().item(0))
      this.map.addLayer(stamentLayer)
    }

    // display() {
    //   this.map.getView().fit([104, 30.6, 104.12, 30.74], this.map.getSize())
    // },
    // moveToLeft() {
    //   var view = this.map.getView()
    //   var mapCenter = view.getCenter()
    //   // 让地图中心的x值增加，即可使得地图向左移动，增加的值根据效果可自由设定
    //   mapCenter[0] += 5000000
    //   view.setCenter(mapCenter)
    //   this.map.render()
    // }
    // addLayer() {
    //   var test = new TileLayer({
    //     source: new XYZ({
    //       url: 'http://t3.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
    //     }),
    //     zIndex: 200
    //   })
    //   this.map.addLayer(test)
    // },
    // addLayer1() {
    //   var test1 = new TileLayer({
    //     source: new XYZ({
    //       url: 'http://t3.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
    //     }),
    //     zIndex: 200
    //   })
    //   this.map.addLayer(test1)
    // }
  }
}
</script>

<style>
  .map{
    height:800px;
    width:100%;
  }
</style>

