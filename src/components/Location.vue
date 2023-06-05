<template>
  <div>
    <h1 class="d-flex justify-content-center">Location Search</h1>
    <div class="container mt-4">
      <div class="d-flex justify-content-center">
        <button
            @click="getCurrentLocation"
            class="btn btn-primary search-button"
        >Get Current Location</button>
        <input class="form-control me-2 search-input ps-2" type="text" v-model="searchInput" @keyup.enter="searchLocation">
        <button class="btn btn-primary search-button" @click="searchLocation">Search</button>
      </div>
    </div>

    <div class="container mt-4">
      <div class="row">
        <div class="col-md-6">
          <div ref="map" style="height: 400px; margin-bottom: 20px;"></div>>
        </div>
        <div class="col-md-6">
          <table class="table-bordered table">
            <thead>
            <tr>
              <th></th>
              <th>Locations</th>
              <th>Time Zone</th>
              <th>Local Time</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(location, index) in paginatedLocations" :key="location.id" :class="{ 'selected': selectedLocations.includes(location.id) }">
              <td><input type="checkbox" @change="toggleSelected(location.id)"></td>
              <td>{{ location.name }}</td>
              <td>{{ location.timezone }}</td>
              <td>{{ location.localTime }}</td>
            </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>



    <div class="container mt-4">
      <div class="d-flex justify-content-between align-content-center">
        <div class="pagination">
          <button class="btn btn-primary pagination-button" :disabled="currentPage === 1" @click="changePage(currentPage - 1)">Previous</button>
          <span class="pagination-info">{{ currentPage }} / {{totalPages}}</span>
          <button class="btn btn-primary pagination-button" :disabled="currentPage === totalPages" @click="changePage(currentPage + 1)">Next</button>
        </div>
      </div>
    </div>
   <div class="container mt-4">
     <button class="btn btn-danger delete-button" @click="deleteSelected" :disabled="selectedLocations.length === 0">Delete Selected</button>
   </div>

  </div>
</template>

<style>
.search-button {
  padding: 8px 12px;
  background-color: #4CAF50;
  border: none;
  border-radius: 4px;
  color: white;
  font-size: 14px;
  cursor: pointer;
}

.search-input {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  width: 200px;
  margin-right: 10px;
}

.pagination-button {
  padding: 8px 12px;
  background-color: #4CAF50;
  border: none;
  border-radius: 4px;
  color: white;
  font-size: 14px;
  cursor: pointer;
  margin-right: 10px;
}

.pagination-info {
  font-size: 14px;
  margin-right: 10px;
}

.delete-button {
  padding: 8px 12px;
  background-color: #f44336;
  border: none;
  border-radius: 4px;
  color: white;
  font-size: 14px;
  cursor: pointer;
}
</style>
<script>
import {Loader} from "@googlemaps/js-api-loader"
export default {
  data(){
    return{
      searchInput: '',
      locations: [],
      selectedLocations: [],
      currentPage: 1,
      itemPerPage: 10,
      map: null,
      markers: [],
    };
  },
  computed: {
    paginatedLocations() {
      const startIndex = (this.currentPage - 1) * this.itemPerPage;
      const endIndex = startIndex + this.itemPerPage;
      return this.locations.slice(startIndex,endIndex);
    },
    totalPages() {
      return Math.ceil(this.locations.length / this.itemPerPage);
    }
  },

  mounted() {
    const loader = new Loader({
      apiKey: 'AIzaSyA9n7LzqrKcRXQHyDgX4HQarhzPp9yL8Qk',
      version: 'weekly',
      libraries: ['places']
    });

    loader.load().then(() => {
        this.map = new google.maps.Map(this.$refs.map, {
          center: {lat: 0, lng: 0},
          zoom: 2,
        });
    }).catch((error) => {
      console.log('Error loading Google Map API', error)
    })
  },

  methods: {
    getCurrentLocation() {
      if (navigator.geolocation){
        navigator.geolocation.getCurrentPosition(this.showPosition);
      }
      else {
        alert('Geolocation is not supported by this browser')
      }
    },

    showPosition(position){
      const lat = position.coords.latitude;
      const lng = position.coords.longitude;
      this.addLocation(lat, lng, google.maps)
    },

    searchLocation() {
      const geocoder = new google.maps.Geocoder();
      geocoder.geocode({ address: this.searchInput }, (results, status) => {
        if (status === 'OK' && results[0]) {
          const location = results[0].geometry.location;
          this.addLocation(location.lat(), location.lng(), google.maps);
        } else {
          alert('Location not found.');
        }
      });
    },

    addLocation(lat, lng, maps) {
      const geocoder = new maps.Geocoder();
      const latLng = new maps.LatLng(lat, lng);
      geocoder.geocode({ location: latLng }, (results, status) => {
        if (status === 'OK' && results[0]) {
          const address = results[0].formatted_address;
          const timezone = results[0].timeZoneId;
          const localTime = new Date().toLocaleTimeString('en-US', { timeZone: timezone });
          const location = {
            id: this.locations.length + 1,
            name: address,
            timezone: timezone,
            localTime: localTime
          };
          this.locations.push(location);
          this.addMarker(lat, lng);
          this.searchInput = '';
        } else {
          alert('Failed to retrieve location details.');
        }
      });
    },

    toggleSelected(locationId) {
      if (this.selectedLocations.includes(locationId)){
        this.selectedLocations = this.selectedLocations.filter(id => id!==locationId)
      }
      else {
        this.selectedLocations.push(locationId)
      }
    },


    changePage(page) {
      if (page >= 1 && page <= this.totalPages){
        this.currentPage = page;
      }
    },

    deleteSelected() {
      this.locations = this.locations.filter(location => !this.selectedLocations.includes(location.id))
      this.clearMarker();
      this.selectedLocations = [];
    },

    clearMarker() {
      this.markers.forEach(marker => marker.setMap(null));
    },

    addMarker(lat, lng) {
      const marker = new google.maps.Marker({
        position: {lat, lng},
        map: this.map
      })
      this.markers.push(marker)
    },
  },

}
</script>