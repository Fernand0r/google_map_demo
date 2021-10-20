<template>
  <div>
    <div class="form">
      <span class="btn" @click="generateCoordinates">Generate</span>
      <input @change="onFileChange($event)" type="file" />
    </div>
    <div class="marker-list">
      <div
        v-for="item in markers"
        :key="item.id"
        @click="setCenter(item.coordinate)"
      >
        <span>
          {{ item.id }}
        </span>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from "@vue/composition-api"
import { openShp } from "shapefile"
export default {
  setup(props, { emit }) {
    const markers = ref([])
    const setCenter = coordinate => {
      emit("change", coordinate)
    }

    const generateLatLng = () => {
      const lat = (Math.random() * (85 - -85 + 1) + -85).toFixed(6)
      const lng = (Math.random() * (180 - -180 + 1) + -180).toFixed(6)

      return { lat, lng }
    }

    const generateCoordinates = () => {
      const cache = []
      let id = markers.value.length
      markers.value.push(
        ...Array.from({ length: 5000 }).map(() => {
          id++
          let lat, lng
          do {
            const { lat: _lat, lng: _lng } = generateLatLng()
            lat = _lat
            lng = _lng
          } while (cache.includes(lat + lng))
          cache.push(lat + lng)
          return {
            coordinate: {
              lat: Number(lat),
              lng: Number(lng),
            },
            id,
          }
        })
      )
      cache.length = 0
      emit("load", markers.value)
    }

    const clearMarkers = () => {
      markers.value = []
    }

    const onFileChange = evt => {
      const files = evt.target.files[0]
      files
        .arrayBuffer()
        .then(openShp)
        .then(source =>
          source.read().then(result => {
            if (result.done) return
            emit("file", result.value)
          })
        )
    }

    return {
      markers,
      setCenter,
      clearMarkers,
      generateCoordinates,
      onFileChange,
    }
  },
}
</script>

<style lang="less" scoped>
@hover-color: rgb(64, 169, 255);
@active-color: rgb(9, 109, 217);
.form {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: flex-start;
  height: 65px;
  margin-bottom: 10px;
}
.btn {
  display: inline-block;
  padding: 5px 10px;
  border-radius: 4px;
  height: 32px;
  font-size: 14px;
  box-sizing: border-box;
  text-align: center;
  cursor: pointer;
  color: #fff;
  border-color: rgb(24, 144, 255);
  background: rgb(24, 144, 255);
  text-shadow: 0 -1px 0 rgb(0 0 0 / 12%);
  box-shadow: 0 2px #0000000b;
  &:hover {
    background-color: @hover-color;
  }
  &:active {
    background-color: @active-color;
  }
}
.upload {
  input {
    display: none;
  }
  button {
    line-height: 1.5715;
    position: relative;
    display: inline-block;
    font-weight: 400;
    white-space: nowrap;
    text-align: center;
    background-image: none;
    border: 1px solid transparent;
    box-shadow: 0 2px #00000004;
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    touch-action: manipulation;
    height: 32px;
    padding: 4px 15px;
    font-size: 14px;
    border-radius: 2px;
    color: #000000d9;
    border-color: #d9d9d9;
    background: #fff;
    cursor: pointer;

    &:hover {
      color: @hover-color;
      border-color: @hover-color;
    }
    &:active {
      color: @active-color;
      border-color: @active-color;
    }
  }
}
.marker-list {
  height: calc(100% - 68px);
  overflow: auto;
  border: 1px solid #ccc;
  &::-webkit-scrollbar {
    width: 10px;
    height: 1px;
  }

  &::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    background: @hover-color;
  }

  &::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    border-radius: 10px;
    background: #ededed;
  }
  div {
    height: 24px;
    border-top: 1px solid #ccc;
    text-align: left;
    padding-left: 20px;
    box-sizing: border-box;
    cursor: pointer;
    &:hover {
      background-color: @hover-color;
      color: #fff;
    }
    &:active {
      background-color: @active-color;
      color: #fff;
    }
    &:first-child {
      border-top: none;
    }
  }
}
</style>
