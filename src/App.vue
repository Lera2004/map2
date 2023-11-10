<template>
  <div class="flex flex-col items-center justify-center h-screen">
    <div class="map-container">
      <div class="map" ref="mapContainer"></div>
      <div class="legend">
        <div class="legend-item">
          <span class="legend-color" style="background-color: green;"></span>
          <span class="legend-text">Здорове</span>
        </div>
        <div class="legend-item">
          <span class="legend-color" style="background-color: red;"></span>
          <span class="legend-text">Хворе</span>
        </div>
        <div class="legend-item">
          <span class="legend-color" style="background-color: orange;"></span>
          <span class="legend-text">Пошкоджене</span>
        </div>
        <div class="legend-item">
          <span class="legend-color" style="background-color: blue;"></span>
          <span class="legend-text">Молоде</span>
        </div>
      </div>
      <label for="treeType" class="text-lg font-bold text-green-600 mt-4">
        Виберіть тип дерева:
      </label>
      <select v-model="selectedType" @change="filterMarkers"
        class="my-select mt-2 p-2 border border-gray-300 rounded w-1/2 md:w-1/4">
        <option value="all">All</option>
        <option v-for="type in uniqueTypes" :value="type" :key="type">
          {{ type }}
        </option>
      </select>
      <label for="treeState" class="text-lg font-bold text-blue-600 mt-4">
        Виберіть стан дерева:
      </label>
      <select v-model="selectedState" @change="filterMarkers"
        class="my-select mt-2 p-2 border border-gray-300 rounded w-1/2 md:w-1/4 ml-2">
        <option value="all">All</option>
        <option v-for="state in uniqueStates" :value="state" :key="state">
          {{ state }}
        </option>
      </select>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import L from "leaflet";
import { treeData } from "./tree_data.js";

const map = ref();
const mapContainer = ref();
const selectedType = ref("all");
const selectedState = ref("all");
let markerLayer = L.layerGroup();

const uniqueTypes = [...new Set(treeData.map(item => item.Tree_Name))];
const uniqueStates = [...new Set(treeData.map(item => item.Tree_State))];

const updateMarkers = () => {
  markerLayer.clearLayers();
  const filteredData = treeData.filter(item =>
    (selectedType.value === "all" || item.Tree_Name === selectedType.value) &&
    (selectedState.value === "all" || item.Tree_State === selectedState.value)
  );

  filteredData.forEach((coordForMarker) => {
    let markerColor = "green";

    switch (coordForMarker.Tree_State) {
      case "Здорове":
        markerColor = "green";
        break;
      case "Хворе":
        markerColor = "red";
        break;
      case "Пошкоджене":
        markerColor = "orange";
        break;
      case "Молоде":
        markerColor = "blue";
        break;
      default:
        markerColor = "green";
    }

    const markerInstance = L.circleMarker(
      [coordForMarker.Latitude, coordForMarker.Longitude],
      {
        data: 'data marker',
        radius: 6,
        fillColor: markerColor,
        color: markerColor
      });
     
    markerInstance.on('click', () => {
    
      onMarkerClick(coordForMarker);
    });

    markerInstance.addTo(markerLayer);

  });
  markerLayer.addTo(map.value);
};

const filterMarkers = () => {
  updateMarkers();
};

const onMarkerClick = (marker) => {
  console.log('Marker clicked:', marker); //тут в нас є вся інформація про дерево, тобі треба додати в template шаблон під картку дерева і тут привлиснити значення потрібним полям (як в тебе було в studentInfo)  
};

onMounted(() => {
  map.value = L.map(mapContainer.value).setView([47.8388, 35.1396], 13);
  L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
  }).addTo(map.value);

  updateMarkers();
});
</script>

<style scoped>
.map {
  width: 1000px;
  height: 600px;
}
.legend {
  margin-left: 16px;
}

.legend-item {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
}

.legend-color {
  width: 20px;
  height: 20px;
  margin-right: 8px;
}
</style>
