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
import { GeoJSON } from "ol/format";
import { fromLonLat } from "ol/proj";

const map = ref(null);

defineOptions({
  name: "IndexPage",
});

// Function to dynamically add storm layers to the map
async function addStormLayers(mapInstance) {
  try {
    const response = await axios.get("http://localhost:3000/storms");
    if (response.data && Array.isArray(response.data.data)) {
      response.data.data.forEach(async (storm) => {
        const detailResponse = await axios.get(
          `http://localhost:3000/storms/${storm.id}/geoInfo`
        );
        const stormLayer = new VectorLayer({
          source: new VectorSource({
            features: new GeoJSON().readFeatures(detailResponse.data, {
              featureProjection: "EPSG:3857",
            }),
          }),
        });
        mapInstance.addLayer(stormLayer);
      });
    } else {
      console.error("Data received is not an array:", response.data);
    }
  } catch (error) {
    console.error("Failed to load storm data:", error);
  }
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
  addStormLayers(map.value);
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
