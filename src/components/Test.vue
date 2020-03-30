<template>
  <div id="map" class="map">
    <form class="form-inline">
      <label>Geometry type &nbsp;</label>
      <select id="type">
        <option value="Point">Point</option>
        <option value="LineString">LineString</option>
        <option value="Polygon">Polygon</option>
      </select>
    </form>
    <!--    <form class="form-inline">-->
    <!--      <label>Measurement type &nbsp;</label>-->
    <!--      <select id="type">-->
    <!--        <option value="length">Length (LineString)</option>-->
    <!--        <option value="area">Area (Polygon)</option>-->
    <!--      </select>-->
    <!--    </form>-->
  </div>
</template>

<script>
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import { Draw, Modify, Snap } from 'ol/interaction'
import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer'
import { OSM, Vector as VectorSource } from 'ol/source'
import { Circle as CircleStyle, Fill, Stroke, Style } from 'ol/style'
// import { unByKey } from 'ol/Observable'
// import Overlay from 'ol/Overlay'
// import { getLength } from 'ol/sphere'
// import { LineString } from 'ol/geom'
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
      // var extent = new GeoJSON().readGeometry(geometry).getExtent()
      // this.map = new Map({
      //   target: 'map',
      //   layers: [
      //     // new TileLayer({
      //     //   source: new XYZ({
      //     //     url: 'http://t4.tianditu.com/DataServer?T=vec_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6' // 图层数据源
      //     //   }),
      //     //   zIndex: 200,
      //     //   visible: true
      //     // })
      //     new TileLayer({
      //       source: new OSM()
      //     }),
      //     new TileLayer({
      //       source: new TileDebug()
      //     })
      //   ],
      //   view: new View({
      //     projection: 'EPSG:4326',
      //     center: [0, 0],
      //     zoom: 1
      //   })
      // })
      // function makeSmooth(path, numIterations) {
      //   numIterations = Math.min(Math.max(numIterations, 1), 10)
      //   while (numIterations > 0) {
      //     path = smooth(path)
      //     numIterations--
      //   }
      //   return path
      // }
      var raster = new TileLayer({
        source: new OSM()
      })

      var source = new VectorSource()
      var vector = new VectorLayer({
        source: source,
        style: new Style({
          fill: new Fill({
            color: 'rgba(255, 255, 255, 0.2)'
          }),
          stroke: new Stroke({
            color: '#ffcc33',
            width: 2
          }),
          image: new CircleStyle({
            radius: 7,
            fill: new Fill({
              color: '#ffcc33'
            })
          })
        })
      })

      var map = new Map({
        layers: [raster, vector],
        target: 'map',
        view: new View({
          center: [-11000000, 4600000],
          zoom: 4
        })
      })

      var modify = new Modify({ source: source })
      map.addInteraction(modify)

      var draw, snap // global so we can remove them later
      var typeSelect = document.getElementById('type')

      function addInteractions() {
        draw = new Draw({
          source: source,
          type: typeSelect.value
        })
        map.addInteraction(draw)
        snap = new Snap({ source: source })
        map.addInteraction(snap)
      }

      /**
       * Handle change event.
       */
      typeSelect.onchange = function() {
        map.removeInteraction(draw)
        map.removeInteraction(snap)
        addInteractions()
      }

      addInteractions()
    }
  }
}
</script>

<style scoped>
#map{
  height:400px;
  width: 100%;
}
</style>
