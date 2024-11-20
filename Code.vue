<template>
  <div class="container">
    <h2 class="title">Captura de Ubicación</h2>
    
    <button class="capture-button" @click="getLocation">Capturar Ubicación Automáticamente</button>

    <form @submit.prevent="fetchLocation" v-if="showManualInput">
      <div>
        <label for="latitude">Latitud:</label>
        <input type="number" v-model="latitude" step="any" required />
      </div>
      <div>
        <label for="longitude">Longitud:</label>
        <input type="number" v-model="longitude" step="any" required />
      </div>
      <button type="submit" class="capture-button">Buscar Ubicación Manualmente</button>
    </form>

    <button class="toggle-button" @click="toggleManualInput">
      {{ showManualInput ? 'Ocultar Entrada Manual' : 'Mostrar Entrada Manual' }}
    </button>

    <div v-if="locations.length > 0" class="location-list">
      <h3>Ubicaciones Capturadas:</h3>
      <ul>
        <li v-for="(location, index) in locations" :key="index" class="location-item">
          {{ location.address }} (Lat: {{ location.latitude }}, Lon: {{ location.longitude }})
        </li>
      </ul>
    </div>
    
    <div v-if="suggestedPlaces.length > 0" class="suggestions">
      <h3>Lugares de Interés:</h3>
      <ul>
        <li v-for="(place, index) in suggestedPlaces" :key="index" class="suggestion-item">
          {{ place.name }} - {{ place.distance }} m
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const latitude = ref(null);
const longitude = ref(null);
const locations = ref([]);
const suggestedPlaces = ref([]);
const showManualInput = ref(false);

const getLocation = async () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(async (position) => {
      const lat = position.coords.latitude;
      const lon = position.coords.longitude;

      const address = await fetchGeocode(lat, lon);
      locations.value.push({ latitude: lat, longitude: lon, address });
      suggestedPlaces.value = await fetchNearbyPlaces(lat, lon);
    }, (error) => {
      console.error(error);
      alert("Error al obtener la ubicación.");
    });
  } else {
    alert("Geolocalización no es soportada en este navegador.");
  }
};

const fetchLocation = async () => {
  if (latitude.value && longitude.value) {
    const lat = parseFloat(latitude.value);
    const lon = parseFloat(longitude.value);
    
    const address = await fetchGeocode(lat, lon);
    locations.value.push({ latitude: lat, longitude: lon, address });
    suggestedPlaces.value = await fetchNearbyPlaces(lat, lon);
  } else {
    alert("Por favor, ingrese una latitud y longitud válidas.");
  }
};

const toggleManualInput = () => {
  showManualInput.value = !showManualInput.value;
};

const fetchGeocode = async (latitude, longitude) => {
  try {
    const response = await axios.get(`https://map-geocoding.p.rapidapi.com/json`, {
      params: { latlng: `${latitude},${longitude}` },
      headers: {
        'x-rapidapi-host': 'map-geocoding.p.rapidapi.com',
        'x-rapidapi-key': '4784ae7760msh407a29ce0fa12dap162b70jsnaf14567f7a9f' // Reemplaza con tu clave de API
      }
    });
    return response.data.results[0]?.formatted_address || 'Dirección no encontrada';
  } catch (error) {
    console.error(error);
    return 'Error al obtener la dirección';
  }
};

const fetchNearbyPlaces = async (latitude, longitude) => {
  try {
    const response = await axios.get(`https://map-geocoding.p.rapidapi.com/nearby`, {
      params: { lat: latitude, lng: longitude },
      headers: {
        'x-rapidapi-host': 'map-geocoding.p.rapidapi.com',
        'x-rapidapi-key': '4784ae7760msh407a29ce0fa12dap162b70jsnaf14567f7a9f' // Reemplaza con tu clave de API
      }
    });
    return response.data.results || [];
  } catch (error) {
    console.error(error);
    return [];
  }
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  color: #333;
}

.capture-button {
  display: block;
  width: 100%;
  padding: 10px;
  margin-bottom: 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.capture-button:hover {
  background-color: #0056b3;
}

.toggle-button {
  display: block;
  width: 100%;
  padding: 10px;
  margin-bottom: 20px;
  background-color: #6c757d;
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.toggle-button:hover {
  background-color: #5a6268;
}

.location-list, .suggestions {
  margin-top: 20px;
}

.location-item, .suggestion-item {
  background: #e9ecef;
  padding: 10px;
  margin: 5px 0;
  border-radius: 5px;
}
</style>
