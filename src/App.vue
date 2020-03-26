<template>
  <div id="app">
    <ol-map />
<!--    <w-m-t-s-test />-->
  </div>

</template>

<script>
import OlMap from './components/OlMap'
// import WMTSTest from './components/WMTS_test'
export default {
  name: 'App',
  components: {
    // WMTSTest
    OlMap
  }
}
</script>
<style>
#app {
  height: 100%;
}
  *{
    padding: 0;
    margin: 0;
  }
  html,body{
    height: 100%;
  }
</style>
