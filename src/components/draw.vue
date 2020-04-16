<template>
  <div>
    <div id="map" class="map" />
    <form class="form-inline">
      <label>Geometry type &nbsp;</label>
      <select id="type">
        <option value="Point">Point</option>
        <option value="LineString">LineString</option>
        <option value="Polygon">Polygon</option>
        <option value="Circle">Circle</option>
        <option value="None">None</option>
      </select>
    </form>
  </div>
</template>

<script>
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import Draw from 'ol/interaction/Draw'
import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer'
import { OSM, Vector as VectorSource } from 'ol/source'
export default {
  name: 'Draw',
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
      var raster = new TileLayer({
        source: new OSM()
      })
      var source = new VectorSource({ wrapX: false })
      var vector = new VectorLayer({
        source: source
      })
      var map = new Map({
        layers: [raster, vector],
        target: 'map',
        view: new View({
          center: [-11000000, 4600000],
          zoom: 4
        })
      })
      var typeSelect = document.getElementById('type')
      var draw // global so we can remove it later
      console.log('1', map.getLayers())
      function addInteraction() {
        var value = typeSelect.value
        if (value !== 'None') {
          draw = new Draw({
            source: source,
            type: typeSelect.value
          })
          map.addInteraction(draw)
        }
      }
      typeSelect.onchange = function() {
        map.removeInteraction(draw)
        addInteraction()
      }

      addInteraction()
    }
  }
}
</script>

<style scoped>
#map{
    width: 100%;
    height: 400px;
}
</style>
