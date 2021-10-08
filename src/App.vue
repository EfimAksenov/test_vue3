<script lang="ts">
import { defineComponent, ref, onMounted, computed } from "vue";
import L from "leaflet";

import { IObject } from "./types/Object";
import _objects from "./data/objects.json";

const access_token =
  "pk.eyJ1IjoiZWZpbWtlIiwiYSI6ImNrdWdsM2FuNjFvM2IzMm10OWozNjBvNnMifQ.lNvBxDIkgldN71I-87avGw";

export default defineComponent({
  setup() {
    const objects = ref<IObject[]>(_objects);
    const selected_object = ref<IObject>();
    const search_query = ref<string>("");
    const filtered_objects = computed(() =>
      objects.value.filter((obj) =>
        search_query.value
          ? obj.name.toLowerCase().startsWith(search_query.value.toLowerCase())
          : true
      )
    );
    const map = ref<L.Map>();

    onMounted(() => {
      map.value = L.map("map");
      map.value.setView([51.505, -0.09], 2);
      L.tileLayer(
        `https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${access_token}`,
        {
          attribution:
            'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: "mapbox/streets-v11",
          tileSize: 512,
          zoomOffset: -1,
          accessToken: access_token,
        }
      ).addTo(map.value);

      objects.value
        .map(({ latitude, longitude }) => L.marker([latitude, longitude]))
        .forEach((m) => m.addTo(map.value as L.Map));
    });

    function zoom_to(object: IObject) {
      const { latitude, longitude } = object;
      selected_object.value = object;
      if (map.value) {
        map.value.setView([latitude, longitude], 10);
      }
    }

    return {
      filtered_objects,
      selected_object,
      search_query,
      zoom_to,
    };
  },
});
</script>

<template>
  <div class="page">
    <div id="map"></div>
    <div class="menu">
      <h3>Объекты</h3>
      <input class="search" placeholder="Поиск" v-model="search_query" />
      <ul class="list">
        <li
          v-for="obj in filtered_objects"
          :key="obj.id"
          @click="zoom_to(obj)"
          :class="{ '--selected': selected_object === obj }"
        >
          {{ obj.name }}
        </li>
      </ul>
    </div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
}

ul,
ul li {
  text-indent: 0;
  list-style-type: none;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  width: 100vw;
  height: 100hv;
}

#map {
  position: absolute;
  z-index: 100;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
}

.menu {
  position: absolute;
  z-index: 200;
  top: 20px;
  bottom: 20px;
  right: 20px;
  display: flex;
  flex-direction: column;
  width: 250px;
  padding: 20px 0;
  background-color: white;
  border-radius: 20px;
}

.menu h3 {
  padding: 0 20px 10px;
}

.search {
  border: none;
  padding: 15px 20px;
  background-color: gainsboro;
}

.list {
  overflow-y: scroll;
}

li {
  padding: 5px 20px;
}

li.--selected {
  background-color: yellow;
}
</style>
