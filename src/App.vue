<template>
  <div class="container">
    <div class="map" ref="mapContainer"></div>
    <div class="select-container">
      <label for="smellType" class="select-label">Виберіть тип сморіду:</label>
      <br>
      <select id="smellType" v-model="selectedType" @change="filterMarkers" class="my-select">
        <option value="Йод">Йод</option>
        <option value="Аміак">Аміак</option>
        <option value="Сірководень">Сірководень</option>
        <option value="Сірка">Сірка</option>
        <option value="Металургійний гар">Металургійний гар</option>
        <option value="Горілий пластик">Горілий пластик</option>
        <option value="Хімія">Хімія</option>
        <option value="Гниль">Гниль</option>
      </select>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import L from "leaflet";
import axios from "axios";

const map = ref();
const mapContainer = ref();
const coordFromServer = ref([]);
const selectedType = ref("all");
let markerLayer = L.layerGroup();

const updateMarkers = () => {
  markerLayer.clearLayers();
  const filteredData = selectedType.value === "all"
    ? coordFromServer.value
    : coordFromServer.value.filter(coord => coord.kind_of_smell === selectedType.value);
  console.log("Координати обраного типу:", filteredData);

  filteredData.forEach((coordForMarker) => {
    L.circleMarker([coordForMarker.latitude, coordForMarker.longitude], {
      radius: 4,
      fillColor: "green",
      color: "green",
    }).addTo(markerLayer);
  });
  markerLayer.addTo(map.value);
};

const filterMarkers = () => {
   console.log("successfully");
  updateMarkers();
};

onMounted(async () => {
  map.value = L.map(mapContainer.value).setView([47.8388, 35.1396], 13);
  L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
    maxZoom: 19,
    attribution:
      '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
  }).addTo(map.value);

  try {
    const response = await axios.get("https://zpspace.com.ua/api/airdata");
    coordFromServer.value = response.data;
    updateMarkers();
  } catch (error) {
    console.error("Помилка отримання даних:", error);
  }
});
</script>

<style scoped>
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
}
.map {
  width: 1000px;
  height: 600px;
}
.my-select {
  width: 200px;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
  margin-left: 50px;
}

.select-label {
  margin-left: 60px;
  font-weight: bold;
  font-size: 18px; 
  color: #1e8000; 
}
</style>