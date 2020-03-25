<template>
  <div id="map">
<!--    <ol-map />-->
  </div>
</template>

<script>
// import OlMap from "./components/OlMap";
import Map from 'ol/Map'
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
import View from 'ol/View'
import {formLonLat} from 'ol/proj.js'

export default {
  name: 'App',
  // components: {
  //   OlMap
  // }
  mounted() {
    var map=new Map({
      target: 'map',//对象指向
      layers: [//图层
              new TileLayer({//定义的是平铺图层
                source: new OSM()//图层源 定义图层映射协议
              })
      ],
      view: new View({//视图
        center: formLonLat([106.8,31.8]),//地图中心
        zoom:4 //缩放层级
      })
    })
  }
}
</script>

<style>
/*#app {*/
/*  height: 100%;*/
/*}*/
/*  *{*/
/*    padding: 0;*/
/*    margin: 0;*/
/*  }*/
/*  html,body{*/
/*    height: 100%;*/
/*  }*/
</style>
