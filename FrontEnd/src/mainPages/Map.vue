<template>
  <div>
    <input type="text" v-model="searchInput" placeholder="Enter a location">
    <button @click="searchLocation">Search</button>
    <div id="map" style="height: 400px;"></div>
    <div v-if="searchResults.length > 0">
      <h2>Search Results:</h2>
      <ul>
        <li v-for="result in searchResults" :key="result.place_id" @click="highlightLocation(result)">
          {{ result.display_name }} ({{ result.lat }}, {{ result.lon }})
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// Define a reactive variable for the search input
const searchInput = ref('');

// Define a reactive variable for the search results
const searchResults = ref([]);

// Variable to store the map instance
let mapInstance = null;
let highlightedMarker = null;

// Function to search for a location
const searchLocation = () => {
  // Add more details to the search query for better precision
  const searchQuery = encodeURIComponent(searchInput.value);
  fetch(`https://nominatim.openstreetmap.org/search?q=${searchQuery}&format=json`)
    .then(response => response.json())
    .then(data => {
      // Update the searchResults variable with the API response
      searchResults.value = data;
      // If there are search results, show the map with markers
      if (data.length > 0) {
        const center = [data[0].lat, data[0].lon];
        showMap(center, data);
      }
    })
    .catch(error => {
      console.error("Error searching location:", error);
    });
};

// Function to show the map with markers for search results
const showMap = (center, searchResults) => {
  // Remove existing map instance if it exists
  if (mapInstance !== null) {
    mapInstance.remove();
  }

  // Initialize the map with center coordinates
  mapInstance = L.map('map').setView(center, 12);

  // Add OpenStreetMap tile layer
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
  }).addTo(mapInstance);

  // Add markers for each search result location
  searchResults.forEach(result => {
    const { lat, lon, display_name } = result;
    const marker = L.marker([lat, lon]).addTo(mapInstance)
      .bindPopup(display_name)
      .openPopup();
    marker.result = result; // Store the result data with the marker
  });
};

// Function to highlight the selected location on the map
const highlightLocation = (result) => {
  // Remove highlight from previous marker
  if (highlightedMarker !== null) {
    highlightedMarker.setStyle({ fillColor: 'blue' });
  }

  // Highlight the selected marker
  const { lat, lon } = result;
  const marker = mapInstance._layers[L.Util.stamp(result.marker)];
  marker.setStyle({ fillColor: 'red' });

  // Store the highlighted marker
  highlightedMarker = marker;
};
</script>

<style>
#map {
  width: 100%;
}
ul {
  list-style-type: none;
  padding: 0;
  cursor: pointer;
}
li {
  margin-bottom: 5px;
  padding: 5px;
  border: 1px solid #ccc;
}
</style>
