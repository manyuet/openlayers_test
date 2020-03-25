<template>
  <div id="app">
    <ol-map />
  </div>
</template>

<script>
import OlMap from "./components/OlMap";

export default {
  name: 'App',
  components: {
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
