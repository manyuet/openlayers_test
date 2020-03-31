<template>
  <div id="map" class="map">
    <!--    <form class="form-inline">-->
    <!--      <label>Geometry type &nbsp;</label>-->
    <!--      <select id="type">-->
    <!--        <option value="Point">Point</option>-->
    <!--        <option value="LineString">LineString</option>-->
    <!--        <option value="Polygon">Polygon</option>-->
    <!--      </select>-->
    <!--    </form>-->
    <div id="mouse-position">
      <span>Projection </span>
    </div>
    <form class="form-inline">
      <label>Measurement type &nbsp;</label>
      <select id="measure">
        <option value="length">Length (LineString)</option>
        <option value="area">Area (Polygon)</option>
      </select>
    </form>
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
import { unByKey } from 'ol/Observable'
import Overlay from 'ol/Overlay'
import { getLength, getArea } from 'ol/sphere'
import { LineString, Polygon } from 'ol/geom'
import { defaults as defaultControls } from 'ol/control'
import MousePosition from 'ol/control/MousePosition'
import { createStringXY } from 'ol/coordinate'
export default {
  name: 'Test',
  data() {
    return {
      mousePositionControl: null
    }
  },
  mounted() {
    this.measure()
    // this.draw()
  },
  methods: {
    draw() {
      var raster = new TileLayer({
        source: new OSM()
      })

      var source = new VectorSource()
      var vector = new VectorLayer({
        source: source,
        style: new Style({
          fill: new Fill({ // 填充样式
            color: 'rgba(255, 255, 255, 0.2)'
          }),
          stroke: new Stroke({ // 描边样式
            color: '#ffcc33',
            width: 2
          }),
          image: new CircleStyle({ // 图像样式
            radius: 7,
            fill: new Fill({
              color: '#ff5658'
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

      var modify = new Modify({ source: source }) // 修改要素几何的交互
      map.addInteraction(modify) // 将给定的互动添加到地图中

      var draw, snap // global so we can remove them later
      var typeSelect = document.getElementById('type')

      function addInteractions() {
        draw = new Draw({ // 绘图要素几何的交互
          source: source, // 绘图要素的目标源
          type: typeSelect.value // 几何类型，拿到option的选择类型
        })
        map.addInteraction(draw)
        snap = new Snap({ source: source }) // 捕捉交互，source：从此来源捕捉功能
        map.addInteraction(snap)
      }

      /**
       * Handle change event.
       */
      typeSelect.onchange = function() { // onchange也可用于单选框与复选框改变后触发的事件
        map.removeInteraction(draw) // 从地图中删除给定的互动
        map.removeInteraction(snap)
        addInteractions()
      }
      addInteractions()
    },
    mousePosition() { // 实时监测鼠标坐标
      this.mousePositionControl = new MousePosition({ // 显示鼠标光标的2D坐标的控件
        coordinateFormat: createStringXY(4), // 坐标格式
        projection: 'EPSG:4326',
        className: 'custom-mouse-position', // css类名称
        target: document.getElementById('mouse-position'), // 使控件在地图视口之外呈现，请指定一个目标。
        undefinedHTML: '&nbsp;' // 标记以显示坐标何时不可用（例如，当指针离开地图视口时）
      })
      // var projectionSelect = document.getElementById('projection')
      // projectionSelect.addEventListener('change', function(event) {
      //   mousePositionControl.setProjection(event.target.value) // 报告鼠标位置的投影。
      // })
      //
      // var precisionInput = document.getElementById('precision')
      // precisionInput.addEventListener('change', function(event) {
      //   var format = createStringXY(event.target.valueAsNumber)
      //   mousePositionControl.setCoordinateFormat(format) // 用于渲染当前位置的坐标格式类型。
      // })
    },
    measure() { // 测量长度，面积
      this.mousePosition()
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

      /**
       * Currently drawn feature.
       * @type {import("../src/ol/Feature.js").default}
       */
      var sketch

      /**
       * The help tooltip element.
       * @type {HTMLElement}
       */
      var helpTooltipElement

      /**
       * Overlay to show the help messages.
       * @type {Overlay}
       */
      var helpTooltip

      /**
       * The measure tooltip element.
       * @type {HTMLElement}
       */
      var measureTooltipElement

      /**
       * Overlay to show the measurement.
       * @type {Overlay}
       */
      var measureTooltip
      /**
       * Handle pointer move.
       * @param {import("../src/ol/MapBrowserEvent").default} evt The event.
       */
      var pointerMoveHandler = function(evt) {
        if (evt.dragging) { // dragging:指示当前是否正在拖动地图。默认值为false,拖动为true
          // if这段代码好像注释掉也没什么影响，好像是为了拖动地图是不触发事件？？
          return
        }
        helpTooltip.setPosition(evt.coordinate) // setPosition(位置)：设置此叠加层的位置
        helpTooltipElement.classList.remove('hidden') // 移除已经存在的类名;
      }

      var map = new Map({
        controls: defaultControls().extend([this.mousePositionControl]),
        layers: [raster, vector],
        target: 'map',
        view: new View({
          center: [-11000000, 4600000],
          zoom: 8
        })
      })
      map.on('pointermove', pointerMoveHandler) // 监听type的pointerMoveHandler
      // addEventListener=onclick
      map.getViewport().addEventListener('mouseout', function() { // getViewport():拿到作为地图viewport的元素
        helpTooltipElement.classList.add('hidden') // 添加新的类
      })

      var typeSelect = document.getElementById('measure')

      var draw // global so we can remove it later

      /**
       * Format length output.
       * @param {LineString} line The line.
       * @return {string} The formatted length.
       */
      var formatLength = function(line) {
        var length = getLength(line)
        var output
        if (length > 100) {
          output = (Math.round(length / 1000 * 100) / 100) +
                  ' ' + 'km'
        } else {
          output = (Math.round(length * 100) / 100) +
                  ' ' + 'm'
        }
        return output // output测量结果
      }

      /**
       * Format area output.
       * @param {Polygon} polygon The polygon.
       * @return {string} Formatted area.
       */
      var formatArea = function(polygon) {
        var area = getArea(polygon)
        var output
        if (area > 10000) {
          output = (Math.round(area / 1000000 * 100) / 100) +
                  ' ' + 'km<sup>2</sup>'
        } else {
          output = (Math.round(area * 100) / 100) +
                  ' ' + 'm<sup>2</sup>'
        }
        return output
      }

      function addInteraction() {
        var type = (typeSelect.value === 'area' ? 'Polygon' : 'LineString')
        draw = new Draw({ // 绘图要素几何的交互
          source: source,
          type: type,
          style: new Style({
            fill: new Fill({
              color: 'rgba(255, 255, 255, 0.2)'
            }),
            stroke: new Stroke({
              color: 'rgba(0, 0, 0, 0.5)',
              lineDash: [10, 10],
              width: 2
            }),
            image: new CircleStyle({
              radius: 5,
              stroke: new Stroke({
                color: 'rgba(0, 0, 0, 0.7)'
              }),
              fill: new Fill({
                color: 'rgba(255, 255, 255, 0.2)'
              })
            })
          })
        })
        map.addInteraction(draw)

        createMeasureTooltip()
        createHelpTooltip()

        var listener
        draw.on('drawstart', // 聆听某种类型的事件。
          function(evt) {
            // set sketch
            sketch = evt.feature

            /** @type {import("../src/ol/coordinate.js").Coordinate|undefined} */
            var tooltipCoord = evt.coordinate
            // getGeometry:获取特征的默认几何
            listener = sketch.getGeometry().on('change', function(evt) {
              var geom = evt.target
              var output
              if (geom instanceof Polygon) {
                output = formatArea(geom)
                tooltipCoord = geom.getInteriorPoint().getCoordinates()
              } else if (geom instanceof LineString) {
                output = formatLength(geom)
                tooltipCoord = geom.getLastCoordinate()
              }
              measureTooltipElement.innerHTML = output
              measureTooltip.setPosition(tooltipCoord) // 设置此叠加层的位置
            })
          })

        draw.on('drawend',
          function() {
            measureTooltipElement.className = 'ol-tooltip ol-tooltip-static'
            measureTooltip.setOffset([0, -7])
            // unset sketch
            sketch = null
            // unset tooltip so that a new one can be created
            measureTooltipElement = null
            createMeasureTooltip()
            unByKey(listener) // 使用on()或返回的键删除事件侦听器once()。
          })
      }

      /**
       * Creates a new help tooltip
       */
      function createHelpTooltip() {
        if (helpTooltipElement) {
          helpTooltipElement.parentNode.removeChild(helpTooltipElement)
        }
        helpTooltipElement = document.createElement('div')
        helpTooltipElement.className = 'ol-tooltip hidden'
        helpTooltip = new Overlay({
          element: helpTooltipElement,
          offset: [15, 0],
          positioning: 'center-left'
        })
        map.addOverlay(helpTooltip)
      }

      /**
       * Creates a new measure tooltip
       */
      function createMeasureTooltip() {
        if (measureTooltipElement) {
          measureTooltipElement.parentNode.removeChild(measureTooltipElement)
        }
        measureTooltipElement = document.createElement('div')
        measureTooltipElement.className = 'ol-tooltip ol-tooltip-measure'
        measureTooltip = new Overlay({
          element: measureTooltipElement,
          offset: [0, -15],
          positioning: 'bottom-center'
        })
        map.addOverlay(measureTooltip)
      }

      /**
       * Let user change the geometry type.
       */
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
  height:400px;
  width: 100%;
}
</style>
