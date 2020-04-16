<template>
  <div id="app">
    <!--    <w-m-t-s-test />-->
    <test />
    <!--      <switch-tian-di-tu />-->
    <!--    <draw />-->
  </div>

</template>

<script>

// import WMTSTest from './components/WMTS_test'
import Test from './components/Test'
// import Draw from './components/draw'
// import SwitchTianDiTu from './components/SwitchTian_Di_tu'

export default {
  name: 'App',
  components: {
    // Draw
    // SwitchTianDiTu
    Test
    // WMTSTest
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
