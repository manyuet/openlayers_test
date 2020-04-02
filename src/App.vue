<template>
  <div id="app">
    <!--    <ol-map />-->
    <!--    <w-m-t-s-test />-->
    <test />
    <!--      <switch-tian-di-tu />-->
  </div>

</template>

<script>
// import OlMap from './components/OlMap'
// import WMTSTest from './components/WMTS_test'
import Test from './components/Test'
// import SwitchTianDiTu from './components/SwitchTian_Di_tu'
export default {
  name: 'App',
  components: {
    // SwitchTianDiTu
    Test
    // WMTSTest
    // OlMap
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
