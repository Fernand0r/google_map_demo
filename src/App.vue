<template>
  <div class="container">
    <Map ref="mapRef" class="map" @clear="clearMarkers" />
    <List
      ref="listRef"
      @change="setCenter"
      @load="generateMarkers"
      @file="loadShapeFile"
      class="list"
    />
  </div>
</template>

<script>
import Map from "./components/Map"
import List from "./components/List"
import { ref } from "@vue/composition-api"

export default {
  name: "App",
  components: {
    Map,
    List,
  },
  setup(props, ctx) {
    const mapRef = ref(null)
    const listRef = ref(null)
    const setCenter = latlng => {
      mapRef.value.centerMarker(latlng)
    }
    const generateMarkers = coordinates => {
      mapRef.value.generateMarkers(coordinates)
    }
    const clearMarkers = () => {
      listRef.value.clearMarkers()
    }
    const loadShapeFile = geoJson => {
      mapRef.value.loadShapeFile(geoJson)
    }

    return {
      mapRef,
      listRef,
      setCenter,
      generateMarkers,
      clearMarkers,
      loadShapeFile,
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
}
body {
  padding: 5px;
  margin: 0;
}
.container {
  height: calc(100vh - 20px);
  display: flex;
  justify-content: space-between;
}
.map {
  width: 70%;
  height: 100%;
}
.list {
  height: 100%;
  width: 28%;
}
</style>
