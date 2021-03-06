<template>
  <div class="LocationPicker">
    <div class="LocationPicker__map" ref="map"></div>
    <input type="text" class="LocationPicker__autocomplete" ref="input"/>
    <info-window class="LocationPicker__info-window" ref="info"></info-window>
  </div>
</template>


<script>
  import InfoWindow from './InfoWindow.vue'
  export default {
    props: ['onPlaceChange'],
    data () {
      return {
        geocoder: null,
        map: null,
        marker: null,
        infoWindow: null,
        autocomplete: null,
          place: {}
      }
    },
    watch:{
        place(){
            if(this.onPlaceChange !== undefined){
                this.onPlaceChange(this.place);
            }
        }
    },
    components: { InfoWindow },

      mounted () {
          this.$parent.$eventHub.$on('location-picker-init', (options) => {

              console.log('location picker map mounted');

              this.geocoder = new google.maps.Geocoder()

              this.map = new google.maps.Map(this.$refs.map, Object.assign({
                  center: { lat: 0, lng: 0 },
                  zoom: 3,
                  disableDefaultUI: true
              }, options.map))

              this.marker = new google.maps.Marker(Object.assign({
                  map: this.map,
                  position: this.map.getCenter(),
                  draggable: true
              }, options.marker))

              this.infoWindow = new google.maps.InfoWindow(Object.assign({
                  content: this.$refs.info.$el
              }, options.infoWindow))

              this.autocomplete = new google.maps.places.Autocomplete(this.$refs.input, Object.assign({
                  types: ['geocode']
              }, options.autocomplete))
              this.map.controls[google.maps.ControlPosition.TOP_LEFT].push(this.$refs.input)

              // events
              google.maps.event.addListenerOnce(this.map, 'idle', this.openInfoWindow)
              this.marker.addListener('dragstart', this.closeInfoWindow)
              this.marker.addListener('dragend', this.geocodeLocation)
              this.autocomplete.addListener('place_changed', this.moveMarker)
          })
      },

    methods: {
      openInfoWindow () {
        this.infoWindow.open(this.map, this.marker)
      },

      closeInfoWindow () {
        this.infoWindow.close()
      },

      geocodeLocation (e) {
        this.map.panTo(e.latLng)
        this.$refs.input.value = ''

        this.geocoder.geocode({'latLng': e.latLng}, (response) => {
          if (response && response.length > 0) {
            this.place = response[0]
            this.$refs.info.showAddress(this.place)
          } else {
            this.place = null
            this.$refs.info.showError()
          }

          this.openInfoWindow()
        })
      },

      moveMarker () {
        var place = this.autocomplete.getPlace()
        var location = place.geometry && place.geometry.location

        if (location) {
          this.place = place
          this.map.panTo(location)
          this.marker.setPosition(location)
          this.$refs.info.showAddress(place)
        }
      }
    }
  }
</script>


<style>
  .LocationPicker,
  .LocationPicker__map {
    height: 100%;
    min-height: 500px;
  }

  .LocationPicker__autocomplete {
    padding: 7px 14px;
    margin: 10px;
    width: 30%;
    min-width: 300px;
    font-family: Roboto;
    font-size: 15px;
    font-weight: 300;
    text-overflow: ellipsis;
    border: 0;
    border-radius: 2px 0 0 2px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
  }

  .LocationPicker > .LocationPicker__autocomplete,
  .LocationPicker > .LocationPicker__info-window {
    display: none;
  }
</style>
