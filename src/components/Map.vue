<template>
  <div ref="mapRef" class="map-wrapper"></div>
</template>

<script>
/* eslint-disable no-unused-vars */
import { onMounted, ref } from "@vue/composition-api"
import { Loader } from "@googlemaps/js-api-loader"

export default {
  setup(props, { emit }) {
    const LngLatOfChengdu = { lat: 30.665855, lng: 104.072242 }
    const mapRef = ref(null)
    const loader = new Loader({
      version: "weekly",
    })
    const maps = ref(null)
    const markers = ref([])
    let map_ins

    const generateMarkers = coordinates => {
      for (let index = 0; index < coordinates.length; index++) {
        const { coordinate: position, id } = coordinates[index]
        markers.value.push({
          marker: new maps.value.Marker({
            position,
            map: map_ins,
            title: String(id),
          }),
          position,
        })
      }
    }

    const centerMarker = latlng => {
      map_ins.setCenter(latlng)
    }

    const initMaps = async () => {
      const { maps: _maps } = await loader.load()
      maps.value = _maps
    }

    const clearMarkers = () => {
      markers.value.forEach(item => {
        item.marker.setMap(null)
      })
      markers.value.length = 0
    }

    const polygons = ref(null)
    const initPolygons = () => {
      if (!markers.value.length) return
      polygons.value = new maps.value.Polygon({
        paths: markers.value.map(({ position }) => position),
        strokeColor: "#FF0000",
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: "#FF0000",
        fillOpacity: 0.35,
      })

      polygons.value.setMap(map_ins)
    }

    const initColorPicker = () => {
      const $btn = document.createElement("div")
      $btn.innerText = "random color picker"
      Object.assign($btn.style, {
        width: "120px",
        height: "32px",
        lineHeight: "32px",
        textAlign: "center",
        index: 1,
        borderRadius: "2px",
        backgroundColor: "#fff",
        margin: "10px",
        cursor: "pointer",
      })

      google.maps.event.addDomListener($btn, "click", function() {
        const currentColor = `#${((Math.random() * 0xffffff) << 0).toString(
          16
        )}`
        polygons.value.setOptions({
          strokeColor: currentColor,
          fillColor: currentColor,
        })
      })

      if (!map_ins.controls[maps.value.ControlPosition.TOP_CENTER].length) map_ins.controls[maps.value.ControlPosition.TOP_CENTER].push($btn)
    }

    const loadShapeFile = geoJson => {
      map_ins.data.addGeoJson({
        type: "FeatureCollection",
        features: [{ type: "Feature", geometry: geoJson }],
      })
      map_ins.data.setStyle({
        strokeColor: "blue",
      })
      const [[[lng, lat], ...rest2], ...rest1] = geoJson.coordinates.flat()
      if (lng && lat) centerMarker({lat, lng})
      map_ins.setZoom(12)
    }

    onMounted(async () => {
      await initMaps()
      map_ins = new maps.value.Map(mapRef.value, {
        center: LngLatOfChengdu,
        zoom: 10,
      })

      map_ins.addListener("rightclick", () => {
        initPolygons()
        clearMarkers()
        initColorPicker()
        emit("clear")
      })
    })

    return {
      mapRef,
      generateMarkers,
      centerMarker,
      loadShapeFile,
    }
  },
}
</script>
