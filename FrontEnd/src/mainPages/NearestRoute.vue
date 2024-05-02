<template>
    <div>
      <input type="text" v-model="destinationInput" placeholder="Enter destination location">
      <div v-for="(input, index) in originInputs" :key="index">
        <input type="text" v-model="originInputs[index]" placeholder="Enter origin location">
      </div>
      <button @click="searchRoute">Search Route</button>
      <div id="map" style="height: 400px;"></div>
      <div>
        <h2>Top 3 Nearest Origin Points:</h2>
        <ul>
          <li v-for="(origin, index) in topThreeOrigins" :key="index">
            {{ origin.name }} (Latitude: {{ origin.lat }}, Longitude: {{ origin.lon }}), Distance: {{ origin.distance.toFixed(2) }} km
          </li>
        </ul>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  import L from 'leaflet';
  import 'leaflet-routing-machine';
  
  // Define a reactive variable for the destination input
  const destinationInput = ref('');
  
  // Define a reactive variable for the origin inputs
  const originInputs = ref(['', '', '', '', '']); // Adjust the number of inputs as needed
  
  // Define a reactive variable for the top three nearest origin points
  const topThreeOrigins = ref([]);
  
  // Variable to store the map instance
  let mapInstance = null;
  
  // Function to calculate distance between two points
  const calculateDistance = (lat1, lon1, lat2, lon2) => {
    const R = 6371; // Radius of the earth in km
    const dLat = (lat2 - lat1) * Math.PI / 180;  // Convert degrees to radians
    const dLon = (lon2 - lon1) * Math.PI / 180;
    const a =
      Math.sin(dLat / 2) * Math.sin(dLat / 2) +
      Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
      Math.sin(dLon / 2) * Math.sin(dLon / 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    const distance = R * c; // Distance in km
    return distance;
  };
  
  // Function to search for a route
  const searchRoute = async () => {
    const destinationQuery = encodeURIComponent(destinationInput.value);
  
    // Filter out empty origin inputs
    const originQueries = originInputs.value.filter(input => input.trim() !== '').map(input => encodeURIComponent(input));
  
    try {
      const destinationResponse = await fetch(`https://nominatim.openstreetmap.org/search?q=${destinationQuery}&format=json`);
      const destinationData = await destinationResponse.json();
  
      if (destinationData.length > 0) {
        const { lat: destLat, lon: destLon } = destinationData[0];
  
        // Calculate distances for all origin points
        const originDistances = [];
        for (const originQuery of originQueries) {
          const originResponse = await fetch(`https://nominatim.openstreetmap.org/search?q=${originQuery}&format=json`);
          const originData = await originResponse.json();
          if (originData.length > 0) {
            const { lat: originLat, lon: originLon, display_name } = originData[0];
            const distance = calculateDistance(originLat, originLon, destLat, destLon);
            originDistances.push({ name: display_name, lat: originLat, lon: originLon, distance });
          }
        }
  
        // Sort origin points by distance
        originDistances.sort((a, b) => a.distance - b.distance);
  
        // Update topThreeOrigins with the nearest three origin points
        topThreeOrigins.value = originDistances.slice(0, 3);
  
        // Display route for the nearest three origin points
        for (const origin of topThreeOrigins.value) {
          showRoute(origin.lat, origin.lon, destLat, destLon);
        }
      }
    } catch (error) {
      console.error("Error searching locations:", error);
    }
  };
  
  // Function to show the route on the map
  const showRoute = (originLat, originLon, destLat, destLon) => {
    // Initialize the map if not yet initialized
    if (mapInstance === null) {
      mapInstance = L.map('map').setView([originLat, originLon], 15);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
      }).addTo(mapInstance);
    }
  
    // Add the route between the origin and destination locations
    L.Routing.control({
      waypoints: [
        L.latLng(originLat, originLon),
        L.latLng(destLat, destLon)
      ],
      routeWhileDragging: true
    }).addTo(mapInstance);
  
    // Add a red marker for the origin location
    L.marker([originLat, originLon], { icon: redIcon }).addTo(mapInstance);
  };
  
  // Define a red icon for the origin markers
  const redIcon = L.icon({
    iconUrl: 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png',
    shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
    iconSize: [25, 41],
    iconAnchor: [12, 41],
    popupAnchor: [1, -34],
    shadowSize: [41, 41]
  });
  </script>
  
  <style>
  #map {
    width: 100%;
  }
  </style>
  