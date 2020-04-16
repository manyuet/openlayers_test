<template>
  <div id="map" class="map">
    <div class="drawMap">
      <select id="type">
        <option value="Point">point</option>
        <option value="LineString">line</option>
        <option value="Polygon">polygon</option>
      </select>
      <button @click="draw">draw</button>
    </div>

    <el-select id="measure" v-model="measureValue" placeholder="请选择" size="mini" style="width: 100px">
      <el-option value="length" />
      <el-option value="area" />
    </el-select>
    <el-button type="primary" circle size="mini" @click="measure">
      <i class="el-icon-crop" />
    </el-button>

    <el-button circle size="mini" @click="changedToImg">影像图层</el-button>
    <el-button circle size="mini" @click="changedToTer">地形图层</el-button>
    <span>路网</span>
    <el-switch v-model="luWangSwitch" @change="changedToVec">路网</el-switch>
    <span>多屏对比</span>
    <el-switch v-model="swipeSwitch" @change="swipe">swipe</el-switch>
    <input v-show="swipeSwitch" id="swipe" type="range" style="width: 100%">
    <div id="mouse-position" />
  </div>
</template>

<script>
import 'ol/ol.css'
import Map from 'ol/Map'
import View from 'ol/View'
import { Draw } from 'ol/interaction'
import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer'
import { OSM, Vector as VectorSource } from 'ol/source'
import { unByKey } from 'ol/Observable'
import Overlay from 'ol/Overlay'
import { getLength, getArea } from 'ol/sphere'
import { LineString, Polygon } from 'ol/geom'
import MousePosition from 'ol/control/MousePosition'
import { createStringXY } from 'ol/coordinate'
import XYZ from 'ol/source/XYZ'
import { fromLonLat } from 'ol/proj'
import { getRenderPixel } from 'ol/render'
export default {
  name: 'Test',
  data() {
    return {
      measureValue: '',
      swipeSwitch: false,
      luWangSwitch: false,
      mousePositionControl: null,
      map: null,
      img: new TileLayer({
        source: new XYZ({
          title: '影像图层',
          url: 'http://t3.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=240859a554196e2374a6bb80cfdd3de6'
        })
      }),
      map_cva: new TileLayer({
        source: new XYZ({
          title: '天地图文字标注',
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
      }),
      drawLayer: new VectorLayer({
        source: new VectorSource()
      }),
      toolLayer: {}
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
      this.map.addLayer(this.drawLayer)
      this.mousePosition()
    }, // 初始化地图
    mousePosition() { // 实时监测鼠标坐标
      this.mousePositionControl = new MousePosition({ // 显示鼠标光标的2D坐标的控件
        coordinateFormat: createStringXY(4), // 坐标格式
        projection: 'EPSG:4326',
        className: 'custom-mouse-position', // css类名称
        target: document.getElementById('mouse-position'), // 使控件在地图视口之外呈现，请指定一个目标。
        undefinedHTML: '&nbsp;' // 标记以显示坐标何时不可用（例如，当指针离开地图视口时）
      })
      this.map.addControl(this.mousePositionControl)
    }, // 鼠标位置
    swipe(map) {
      var aerial = new TileLayer({
        source: new XYZ({
          // attributions: attributions,
          url: 'http://t3.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=d0cf74b31931aab68af181d23fa23d8d',
          maxZoom: 20
        })
      })
      map = this.map
      console.log(this.map.getLayers())
      if (this.swipeSwitch) {
        map.getLayers().insertAt(1, aerial)
        var swipe = document.getElementById('swipe') // 用于控制卷帘位置的DOM元素

        aerial.on('prerender', (event) => { // 在渲染之前触发
          var ctx = event.context // 获得canvas渲染上下文
          var mapSize = map.getSize() // 地图的width,height
          var width = mapSize[0] * (swipe.value / 100) // 用于保存卷帘的位置(卷帘宽度)
          var tl = getRenderPixel(event, [width, 0]) // 返回值是event的canvas的像素
          var tr = getRenderPixel(event, [mapSize[0], 0])
          var bl = getRenderPixel(event, [width, mapSize[1]])
          var br = getRenderPixel(event, mapSize)
          ctx.save() // 保存canvas设置
          ctx.beginPath() // 开始绘制路径
          ctx.moveTo(tl[0], tl[1])
          ctx.lineTo(bl[0], bl[1])
          ctx.lineTo(br[0], br[1])
          ctx.lineTo(tr[0], tr[1])
          ctx.closePath()
          ctx.clip() // 裁剪aerial地图，以形成卷帘效果
        })

        aerial.on('postrender', (event) => { // 在aerial地图渲染之后触发
          var ctx = event.context
          ctx.restore() // 恢复canvas设置
        })

        swipe.addEventListener('input', () => { // 在每次用户改变swipe控件时触发
          map.render() // 渲染地图
        }, false)
      } else {
        map.removeLayer(this.map.getLayers().item(1))
      }
    }, // 卷帘效果
    changedToImg() {
      this.map.removeLayer(this.map.getLayers().item(0))
      var layersArray = this.map.getLayers()
      layersArray.insertAt(0, this.img)
      console.log(this.map.getLayers().array_)
    }, // 切换成影像图层
    changedToVec() {
      var layersArray = this.map.getLayers()
      if (this.luWangSwitch) {
        layersArray.insertAt(1, this.map_cva)
      } else {
        this.map.removeLayer(this.map_cva)
      }
    }, // 路网开关
    changedToTer() {
      this.map.removeLayer(this.map.getLayers().item(0))
      var layersArray = this.map.getLayers()
      layersArray.insertAt(0, this.map_ter)
    }, // 切换成地形图层
    draw(map, drawLayer) {
      map = this.map
      drawLayer = this.drawLayer
      var typeSelect = document.getElementById('type')

      var draw // global so we can remove it later
      function addInteraction() {
        draw = new Draw({
          source: map.getLayers().array_[1].getSource(),
          type: typeSelect.value
        })
        map.getLayers().array_[1].getSource().clear()
        map.addInteraction(draw)
      }
      typeSelect.onchange = function() {
        // map.getLayers().array_[1].getSource().clear()
        map.removeInteraction(draw)
        addInteraction()
      }
      addInteraction()
    },
    measure(map, vector) { // 测量长度，面积
      map = this.map
      vector = this.drawLayer
      var sketch
      var helpTooltipElement // 帮助提示框对象
      var helpTooltip // 帮助提示信息对象
      var measureTooltipElement // 测量提示框对象
      var measureTooltip // 测量提示信息对象
      var listener
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
      var pointerMoveHandler = function(evt) { // 鼠标移动触发的函数
        if (evt.dragging) { // dragging:指示当前是否正在拖动地图。默认值为false,拖动为true
          // if这段代码好像注释掉也没什么影响，好像是为了拖动地图是不触发事件？？
          return
        }
        helpTooltip.setPosition(evt.coordinate) // setPosition(位置)：设置此叠加层的位置
        helpTooltipElement.classList.remove('hidden') // 移除已经存在的类名;当鼠标移除地图视图的时为帮助提示要素添加隐藏样式
      }
      var typeSelect = document.getElementById('measure')
      var drawType = (typeSelect.value === 'area' ? 'Polygon' : 'LineString')
      function addInteraction() {
        draw = new Draw({ // 绘图要素几何的交互
          source: vector.getSource(),
          type: drawType
        })
        map.getLayers().array_[1].getSource().clear()
        map.addInteraction(draw)
        createMeasureTooltip()
        createHelpTooltip()
        // 在绘图开始时实时计算当前当前绘制线的长度或多边形的面积，以提示框形式显示
        draw.on('drawstart', // 聆听type的事件。
          function(evt) {
            // set sketch
            sketch = evt.feature // 绘制的要素

            /** @type {import("../src/ol/coordinate.js").Coordinate|undefined} */
            var tooltipCoord = evt.coordinate // 绘制的坐标
            // getGeometry:获取特征的默认几何
            // 绑定change事件，根据绘制几何类型得到测量长度值或面积值，并将其设置到测量工具提示框中显示
            listener = sketch.getGeometry().on('change', function(evt) {
              var geom = evt.target // 绘制几何要素
              var output
              if (geom instanceof Polygon) {
                output = formatArea(geom)
                tooltipCoord = geom.getInteriorPoint().getCoordinates()
              } else if (geom instanceof LineString) {
                output = formatLength(geom)
                tooltipCoord = geom.getLastCoordinate()
              }
              measureTooltipElement.innerHTML = output // 将测量值设置到测量工具提示框中显示

              measureTooltip.setPosition(tooltipCoord) // 设置测量工具提示框的显示位置,setPosition():设置此叠加层的位置
            })
          })

        // 绘图结束时重新创建一个测量提示框显示测量结果
        draw.on('drawend',
          function() {
            measureTooltipElement.className = 'ol-tooltip ol-tooltip-static' // 设置测量提示框的样式
            measureTooltip.setOffset([0, -7])
            // unset sketch
            sketch = null // 置空当前绘制的要素对象
            // unset tooltip so that a new one can be created
            measureTooltipElement = null // 置空测量工具提示框对象
            createMeasureTooltip() // 重新创建一个测试工具提示框显示结果
            unByKey(listener) // 使用on()或返回的键删除事件侦听器once()。
            map.un('pointermove', pointerMoveHandler) // 只有一次测量
            map.removeInteraction(draw)
            helpTooltipElement.classList.add('hidden')
          })
      }
      map.on('pointermove', pointerMoveHandler) // 监听type的pointerMoveHandler
      // addEventListener=onclick
      map.getViewport().addEventListener('mouseout', function() { // getViewport():拿到作为地图viewport的元素
        helpTooltipElement.classList.add('hidden') // 添加新的类
      })
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
      typeSelect.onchange = function() {
        map.removeInteraction(draw)
        addInteraction()
      }
      addInteraction()
    } // 测量长度面积
  }
}
</script>

<style scoped>
  #map {
    width: 100%;
    height: 600px;
    position: absolute;
  }
  #mouse-position {
    position: absolute;
    bottom: 5px;
    height: 20px;
    right: 20px;
    /* 将z-index设置为显示在地图上层 */
    z-index: 2000;
    color: aliceblue;
  }
  .drawMap {
    position: absolute;
    background-color: aliceblue;
    right: 20px;
    height: 30px;
  }
</style>
