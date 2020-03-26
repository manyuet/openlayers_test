<template>
  <div id="map" />
</template>

<script>
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import { getWidth, getTopLeft } from 'ol/extent'
import TileLayer from 'ol/layer/Tile'
import { get as getProjection } from 'ol/proj'
// import OSM from 'ol/source/OSM'
import WMTS from 'ol/source/WMTS'
import WMTSTileGrid from 'ol/tilegrid/WMTS'
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
      const projection = getProjection('EPSG:4326')
      const projectionExtent = projection.getExtent()
      const size = getWidth(projectionExtent) / 256
      const resolutions = new Array(14)
      const matrixIds = new Array(14)
      for (var z = 0; z < 14; ++z) {
        // generate resolutions and matrixIds arrays for this WMTS
        resolutions[z] = size / Math.pow(2, z)
        matrixIds[z] = z
      }

      this.map = new Map({
        layers: [
          new TileLayer({
            source: new WMTS({
              url: '240859a554196e2374a6bb80cfdd3de6',
              layer: 'vec',
              matrixSet: 'EPSG:3857',
              format: 'titles',
              projection: projection,
              tileGrid: new WMTSTileGrid({
                origin: getTopLeft(projectionExtent),
                resolutions: resolutions,
                matrixIds: matrixIds
              }),
              style: 'default',
              wrapX: true
            })
          }),
          new TileLayer({
            source: new WMTS({
              url: '240859a554196e2374a6bb80cfdd3de6',
              layer: 'cva',
              style: 'default',
              matrixSet: 'c',
              format: 'titles',
              wrapX: true,
              tileGrid: new WMTSTileGrid({
                origin: getTopLeft(projectionExtent),
                resolutions: resolutions,
                matrixIds: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
              })
            })
          })
        ],
        target: 'map',
        view: new View({
          center: [118.7971, 31.9332], // 中心点
          projection: projection,
          zoom: 11,
          maxZoom: 15,
          minZoom: 1
        })
      })
    }
  }
}
</script>

<style scoped>

</style>
