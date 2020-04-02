<template>
  <div id="map" class="map">
    <el-row>
      <el-col :span="5">
        <el-select id="type" v-model="drawValue" placeholder="请选择" size="mini" style="width: 100px">
          <el-option value="Point" />
          <el-option value="LineString" />
          <el-option value="Polygon" />
        </el-select>
        <el-button type="primary" circle size="mini" @click="draw">
          <i class="el-icon-edit" />
        </el-button>
      </el-col>
      <el-col :span="5">
        <el-select id="measure" v-model="measureValue" placeholder="请选择" size="mini" style="width: 100px">
          <el-option value="length" />
          <el-option value="area" />
        </el-select>
        <el-button type="primary" circle size="mini" @click="measure">
          <i class="el-icon-crop" />
        </el-button>
      </el-col>
      <el-button circle size="mini" @click="changedToImg">Img</el-button>
      <el-button circle size="mini" @click="changedToVec">Vec</el-button>
      <el-button circle size="mini" @click="changedToTer">Ter</el-button>
    </el-row>
    <div id="mouse-position">
      <span>Projection </span>
    </div>
  </div>
</template>

<script>
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import { Draw } from 'ol/interaction'
import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer'
import { OSM, Vector as VectorSource } from 'ol/source'
import { Circle as CircleStyle, Fill, Stroke, Style } from 'ol/style'
import { unByKey } from 'ol/Observable'
import Overlay from 'ol/Overlay'
import { getLength, getArea } from 'ol/sphere'
import { LineString, Polygon } from 'ol/geom'
// import { defaults as defaultControls } from 'ol/control'
import MousePosition from 'ol/control/MousePosition'
import { createStringXY } from 'ol/coordinate'
import XYZ from 'ol/source/XYZ'
import { fromLonLat } from 'ol/proj'
// import Control from 'ol/control/Control'
export default {
  name: 'Test',
  data() {
    return {
      drawValue: '',
      measureValue: '',
      mousePositionControl: null,
      map: null,
      img: new TileLayer({
        source: new XYZ({
          url: 'http://t3.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
        })
      }),
      map_cva: new TileLayer({
        source: new XYZ({
          url: 'http://t3.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
        })
      }),
      map_vec: new TileLayer({
        source: new XYZ({
          url: 'http://t4.tianditu.com/DataServer?T=vec_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
        })
      }),
      map_ter: new TileLayer({
        source: new XYZ({
          url: 'http://t4.tianditu.com/DataServer?T=ter_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
        })
      }),
      map_ter_cva: new TileLayer({
        source: new XYZ({
          url: 'http://t4.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
        })
      })

    }
  },
  mounted() {
    this.init()
    // this.measure()
    // this.draw()
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
      this.mousePosition()
    },
    mousePosition() { // 实时监测鼠标坐标
      this.mousePositionControl = new MousePosition({ // 显示鼠标光标的2D坐标的控件
        coordinateFormat: createStringXY(4), // 坐标格式
        projection: 'EPSG:4326',
        className: 'custom-mouse-position', // css类名称
        target: document.getElementById('mouse-position'), // 使控件在地图视口之外呈现，请指定一个目标。
        undefinedHTML: '&nbsp;' // 标记以显示坐标何时不可用（例如，当指针离开地图视口时）
      })
      this.map.addControl(this.mousePositionControl)
    },
    changedToImg() {
      // layersArray.insertAt(1, this.img)
      this.map.removeLayer(this.map.getLayers().item(0))
      this.map.addLayer(this.img)
    },
    changedToVec() {
      this.map.removeLayer(this.map.getLayers().item(0))
      this.map.addLayer(this.map_vec)
      this.map.addLayer(this.map_cva)
    },
    changedToTer() {
      this.map.removeLayer(this.map.getLayers().item(0))
      this.map.addLayer(this.map_ter)
      this.map.addLayer(this.map_ter_cva)
    },
    draw(map) {
      this.measureValue = ''
      map = this.map
      var drawLayer = new VectorLayer({
        source: new VectorSource()
      })
      map.addLayer(drawLayer)
      var typeSelect = document.getElementById('type')
      var draw
      function addInteraction() {
        draw = new Draw({
          source: drawLayer.getSource(),
          type: typeSelect.value
        })
        map.addInteraction(draw)
      }
      typeSelect.onchange = function() {
        map.removeInteraction(draw)
        addInteraction()
      }
      addInteraction()
    },
    measure(map) { // 测量长度，面积
      this.drawValue = ''
      var source = new VectorSource()
      // setPosition(tooltipCoord)
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

      var sketch

      var helpTooltipElement // 帮助提示框对象

      var helpTooltip // 帮助提示信息对象

      var measureTooltipElement // 测量提示框对象

      var measureTooltip // 测量提示信息对象

      var pointerMoveHandler = function(evt) { // 鼠标移动触发的函数
        if (evt.dragging) { // dragging:指示当前是否正在拖动地图。默认值为false,拖动为true
          // if这段代码好像注释掉也没什么影响，好像是为了拖动地图是不触发事件？？
          return
        }
        helpTooltip.setPosition(evt.coordinate) // setPosition(位置)：设置此叠加层的位置
        helpTooltipElement.classList.remove('hidden') // 移除已经存在的类名;当鼠标移除地图视图的时为帮助提示要素添加隐藏样式
      }
      map = this.map
      map.addLayer(vector)
      map.on('pointermove', pointerMoveHandler) // 监听type的pointerMoveHandler
      // addEventListener=onclick
      map.getViewport().addEventListener('mouseout', function() { // getViewport():拿到作为地图viewport的元素
        helpTooltipElement.classList.add('hidden') // 添加新的类
      })

      var typeSelect = document.getElementById('measure')

      var draw

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
        draw.on('drawstart', // 聆听type的事件。
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
        helpTooltipElement = document.createElement('span')
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
        measureTooltipElement = document.createElement('span')
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
