<template>
  <q-page class="page-container">
    <div id="map" class="map"></div>
  </q-page>
</template>

<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";
import Map from "ol/Map";
import View from "ol/View";
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
import VectorLayer from "ol/layer/Vector";
import VectorSource from "ol/source/Vector";
import KML from "ol/format/KML";
import { fromLonLat } from "ol/proj";

const map = ref(null);

defineOptions({
  name: "IndexPage",
});

// Function to add hurricane data to the map
async function addHurricaneLayers(mapInstance) {
  const response = await axios.get("http://localhost:3000/tropical");
  response.data.forEach((hurricane) => {
    if (hurricane.type != "WindSpeedProbability") {
      hurricane.links.forEach((link) => {
        if (link.href.endsWith(".kmz")) {
          const vectorLayer = new VectorLayer({
            source: new VectorSource({
              url: link.href,
              format: new KML(),
            }),
          });
          mapInstance.addLayer(vectorLayer);
        }
      });
    }
  });
}

onMounted(() => {
  map.value = new Map({
    target: "map",
    layers: [
      new TileLayer({
        source: new OSM(),
      }),
    ],
    view: new View({
      center: fromLonLat([-80.0, 15.0]),
      zoom: 6,
    }),
  });
  addHurricaneLayers(map.value);
});
</script>

<style scoped>
.page-container {
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.map {
  width: 100%;
  height: 100%;
}
</style>
