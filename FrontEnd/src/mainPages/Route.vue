<template>
    <div>
      <input type="text" v-model="originInput" placeholder="Enter origin location">
      <input type="text" v-model="destinationInput" placeholder="Enter destination location">
      <button @click="searchRoute">Search Route</button>
      <div id="map" style="height: 400px;"></div>
      <div v-if="searchResults.length > 0">
        <h2>Search Results:</h2>
        <ul>
          <li v-for="result in searchResults" :key="result.place_id">
            {{ result.display_name }} ({{ result.lat }}, {{ result.lon }})
          </li>
        </ul>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  import L from 'leaflet';
  import 'leaflet-routing-machine';
  
  
  // Define reactive variables for the origin and destination inputs
  const originInput = ref('');
  const destinationInput = ref('');
  
  // Define a reactive variable for the search results
  const searchResults = ref([]);
  
  // Variable to store the map instance
  let mapInstance = null;
  
  // Function to search for a route
  const searchRoute = async () => {
    const originQuery = encodeURIComponent(originInput.value);
    const destinationQuery = encodeURIComponent(destinationInput.value);
  
    try {
      const originResponse = await fetch(`https://nominatim.openstreetmap.org/search?q=${originQuery}&format=json`);
      const originData = await originResponse.json();
  
      const destinationResponse = await fetch(`https://nominatim.openstreetmap.org/search?q=${destinationQuery}&format=json`);
      const destinationData = await destinationResponse.json();
  
      if (originData.length > 0 && destinationData.length > 0) {
        const { lat: originLat, lon: originLon } = originData[0];
        const { lat: destLat, lon: destLon } = destinationData[0];
        showRoute(originLat, originLon, destLat, destLon);
      }
    } catch (error) {
      console.error("Error searching location:", error);
    }
  };
  
  
  // Function to show the route on the map
  const showRoute = (originLat, originLon, destLat, destLon) => {
    // Remove existing map instance if it exists
    if (mapInstance !== null) {
      mapInstance.remove();
    }
  
    // Initialize the map with center coordinates
    mapInstance = L.map('map').setView([originLat, originLon], 15);
  
    // Add OpenStreetMap tile layer
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(mapInstance);
  
    // Add the route between the origin and destination locations
    L.Routing.control({
      waypoints: [
        L.latLng(originLat, originLon),
        L.latLng(destLat, destLon)
      ],
      routeWhileDragging: true
    }).addTo(mapInstance);
  };
  </script>
  
  <style>
  #map {
    width: 100%;
  }
  </style>
  