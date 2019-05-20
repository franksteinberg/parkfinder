<template>
  <div>
    <div style="display: none">
      <div id="info-content" />
    </div>
    <div>
      <label>
        <input
          placeholder="Search"
          ref="autocomplete"
          class="search-location block w-full sm:w-1/2 mx-auto mt-3 p-2 text-lg leading-relaxed rounded shadow shadow-lg"
          onfocus="value = ''" 
          type="text"
          @place_changed="setPlace"
        />
      </label>
    </div>
    <div v-show="currentPlace" class="flex flex-col lg:flex-row block">
      <div class="google-map flex-grow mx-auto m-4 w-full bg-gray-400 shadow shadow-lg" :id="mapName"></div>
      <div class="w-full lg:w-2/5 lg:m-4 lg:pl-3 lg:pr-6 lg:h-65vh lg:overflow-y-auto">
        Park Results
      </div>
    </div>
  </div>
</template>

<script>
import gmapsInit from '../utils/gmaps';

export default {
  name: "App",
  props: ['name'],

  data() {
    return {
      markers: [],
      parks: [],
      currentPlace: null,
      mapName: this.name + "-map",
      map: null,
      autocomplete: null,
      places: null,
      infoWindow: null,
    };
  },

  async mounted() {
    try {
      const google = await gmapsInit();
      const geocoder = new google.maps.Geocoder();
      this.map = new google.maps.Map(this.$el.getElementsByClassName('google-map')[0]);

      geocoder.geocode({ address: 'Norfolk, VA' }, (results, status) => {
        if (status !== 'OK' || !results[0]) {
          throw new Error(status);
        }

        this.map.setCenter(results[0].geometry.location);
        this.map.fitBounds(results[0].geometry.viewport);
      });

      this.autocomplete = new window.google.maps.places.Autocomplete(
        (this.$refs.autocomplete),
        {types: ['geocode']}
      );
      this.places = new google.maps.places.PlacesService(this.map);
      this.infoWindow = new google.maps.InfoWindow({
        content: document.getElementById('info-content')
      });

      this.autocomplete.addListener('place_changed', () => {
        this.infoWindow.close()
        
        this.setPlace(this.autocomplete.getPlace())
      });
    } catch (error) {
      console.error(error);
    }
  },

  methods: {
    // receives a place object via the autocomplete component
    setPlace(place) {
      this.currentPlace = place;
      if (place.geometry) {
        this.map.panTo(place.geometry.location);
        this.map.setZoom(15);
        this.search();
      }
    },
    addMarker() {
      if (this.currentPlace) {
        const marker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng()
        };
        this.markers.push({ position: marker });
        this.places.push(this.currentPlace);
        this.center = marker;
        this.currentPlace = null;
      }
    },
    search() {
      var search = {
        location: {lat: this.currentPlace.geometry.location.lat(),lng: this.currentPlace.geometry.location.lng()},
        radius: 500,
        types: ['park']
      };

      this.places.nearbySearch(search, (results, status) => {
        if (status === google.maps.places.PlacesServiceStatus.OK) {
          this.parks = results
          this.clearMarkers()

          // Create a marker for each park found, and
          for (var i = 0; i < this.parks.length; i++) {
            this.parks[i].resultNumber = i + 1
            var markerIcon = 'https://raw.githubusercontent.com/Concept211/Google-Maps-Markers/master/images/marker_green' 
              + this.parks[i].resultNumber + '.png';
            // Use marker animation to drop the icons incrementally on the map.
            this.markers[i] = new google.maps.Marker({
              position: this.parks[i].geometry.location,
              animation: google.maps.Animation.DROP,
              icon: markerIcon
            });

            // If the user clicks a marker, show the details
            // in an info window.

            var content = `
              <div class="w-full bg-white mb-1 flex items-center text-green-800">
                <div class="h-full w-12 border-r border-grey-400 text-center flex items-center">
                  <div class="flex-1 opacity-75 text-3xl">${this.parks[i].resultNumber}</div>
                </div>
                <div class="h-full flex-grow flex items-center p-2">
                  <div class="text-lg flex-1 my-4 font-medium">
                    <span class="block uppercase">${this.parks[i].name}</span>
                  </div>
                </div>
              </div>
            `

            google.maps.event.addListener(this.markers[i], 'click', ((marker,content,infowindow) => { 
              return () => {
                  infowindow.setContent(content);
                  infowindow.open(this.map,marker);
              };
            })(this.markers[i],content,this.infoWindow));  
            
            setTimeout(this.dropMarker(i), i * 100);
          }
          return
        }
      });
    },
    dropMarker(i) {
      return () => {
        this.markers[i].setMap(this.map);
      }
    },
    clearMarkers() {
      for (var i = 0; i < this.markers.length; i++) {
        if (this.markers[i]) {
          this.markers[i].setMap(null);
        }
      }
      this.markers = [];
    },
  },
};
</script>

<style scoped>
  .google-map {
    height: 65vh;
  }
</style>
