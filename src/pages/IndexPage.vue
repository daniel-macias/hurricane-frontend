<template>
  <q-page class="page-container">
    <div id="map" class="map"></div>
  </q-page>
</template>

<script setup>
import { onMounted, ref, inject, watch } from "vue";
import axios from "axios";
import Map from "ol/Map";
import View from "ol/View";
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
import VectorLayer from "ol/layer/Vector";
import VectorSource from "ol/source/Vector";
import { GeoJSON } from "ol/format";
import { fromLonLat } from "ol/proj";
import { Style, Stroke } from "ol/style";

const map = ref(null);
const showHurricanes = inject("showHurricanes"); // Inject the shared state
const hurricaneLayers = ref([]);

defineOptions({
  name: "IndexPage",
});
// Adjust visibility of all hurricane layers
function updateHurricaneVisibility() {
  hurricaneLayers.value.forEach((layer) => {
    layer.setVisible(showHurricanes.value);
  });
}

watch(showHurricanes, () => {
  updateHurricaneVisibility();
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
          style: function (feature) {
            const geometryType = feature.getGeometry().getType();
            const properties = feature.getProperties();
            console.log(`Feature Type: ${geometryType}`);
            console.log(`Properties:`, properties);
            // Basic styling based on the feature type
            return new Style({
              stroke: new Stroke({
                color: properties.color || "#ffcc33", // Use property color or a default one
                width: 4,
                lineCap: "round", // Makes the ends of the lines rounded
                lineJoin: "bevel", // Makes the joins between lines beveled
                lineDash: [15, 5], // Dash pattern: 15 pixels filled, 5 pixels empty
                lineDashOffset: 2, // Starts the dash pattern with an offset
                opacity: 0.75, // Semi-transparent
              }),
            });
          },
        });
        mapInstance.addLayer(stormLayer);
        hurricaneLayers.value.push(stormLayer); // Add the layer to the hurricaneLayers list
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
