<template>
  <div>
    <div id="map" class="map" />
    <button @click="changed_img">切换影像底图</button>
    <button @click="changed_vec">切换街道底图</button>
    <button @click="changed_ter">切换地形底图</button>
  </div>
</template>

<script>
import 'ol/ol.css'
import { Map, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
import XYZ from 'ol/source/XYZ'
import { fromLonLat } from 'ol/proj'
export default {
  name: 'SwitchTianDiTu',
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
            source: new OSM()
          })
        ],
        view: new View({
          center: fromLonLat([116.24, 39.55]),
          zoom: 4
        })
      })
    },
    changed_img() {
      var img = new TileLayer({
        source: new XYZ({
          url: 'http://t3.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=d0cf74b31931aab68af181d23fa23d8d'
        })
      })
      this.map.addLayer(img)
    },
    changed_vec() {
      var map_cva = new TileLayer({
        source: new XYZ({
          url: 'http://t3.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=d0cf74b31931aab68af181d23fa23d8d'
        })
      })
      var map_vec = new TileLayer({
        source: new XYZ({
          url: 'http://t4.tianditu.com/DataServer?T=vec_w&x={x}&y={y}&l={z}&tk=d0cf74b31931aab68af181d23fa23d8d'
        })
      })
      this.map.addLayer(map_vec)
      this.map.addLayer(map_cva)
    },
    changed_ter() {
      var map_ter = new TileLayer({
        source: new XYZ({
          url: 'http://t4.tianditu.com/DataServer?T=ter_w&x={x}&y={y}&l={z}&tk=d0cf74b31931aab68af181d23fa23d8d'
        })
      })
      var map_cta = new TileLayer({
        source: new XYZ({
          url: 'http://t4.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=d0cf74b31931aab68af181d23fa23d8d'
        })
      })
      this.map.addLayer(map_ter)
      this.map.addLayer(map_cta)
    }

  }
}
</script>

<style scoped>
#map{
    height: 400px;
    width: 100%;
}
</style>
