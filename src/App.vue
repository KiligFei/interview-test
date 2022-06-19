<template>
  <div id="app">
    <MapContainer />
  </div>
</template>

<script>
import MapContainer from './components/MapContainer.vue'

export default {
  name: 'App',
  components: {
    MapContainer
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
