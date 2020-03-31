<template>
  <div id="map" class="map" />
</template>

<script>
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import { Image as ImageLayer, Tile as TileLayer } from 'ol/layer'
import ImageWMS from 'ol/source/ImageWMS'
import OSM from 'ol/source/OSM'
export default {
  name: 'WMTSTest',
  data() {
    return {
      map: null
    }
  },
  mounted() {
    this.test()
  },
  methods: {
    test() {
      var layers = [
        new TileLayer({
          source: new OSM()
        }),
        new ImageLayer({
          extent: [-13884991, 2870341, -7455066, 6338219],
          source: new ImageWMS({ // 提供单幅图像的WMS服务器的源
            url: 'https://ahocevar.com/geoserver/wms',
            params: { 'LAYERS': 'topp:states' },
            ratio: 1, // 1表示图像请求是地图视口的大小，是地图视口2的宽度和高度的两倍，依此类推。必须为1或更高。
            serverType: 'geoserver' // 类型的远程的WMS服务器：mapserver，geoserver或qgis
          })
        })
      ]
      this.map = new Map({
        layers: layers,
        target: 'map',
        view: new View({
          center: [-10997148, 4569099],
          zoom: 4
        })
      })
    }
  }
}
</script>

<style scoped>

</style>
