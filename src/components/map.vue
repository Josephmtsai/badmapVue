<template>
  <div align="center" class="text-center col-12">
    <div align="center" v-if="loading" class="col-12 overlay">
      <hour-glass size="500px"></hour-glass>
    </div>
    <div class="col-12" v-if="!loading">
      <div class="col-4">
        <date-picker v-model="endTime" range></date-picker>
        {{endTime}}
        <span class="label label-default" for="selectLocation">Specify Location</span>
        <div >
          <b-form-select v-model="selected"  id="selectLocation" value-field="name" text-field="name"  :options="locationList"  class="mb-3" ></b-form-select>
        </div>
        <button class="danger" @click="showTodayInfo()" >Show Today </button>
      </div>
      <div class="col-4">
        {{center}}
      </div>      
      <gmap-map  class="col-12" style="height:800px" :center="center"  :zoom="zoom"  map-type-id="roadmap"  >
        <gmap-cluster >
            <gmap-marker :key="index" v-for="(m, index) in markers" :position="m.position" :clickable="true" @click="showModal(m)" :label="m.infoText"  ></gmap-marker>
        </gmap-cluster>
      </gmap-map>

      <b-modal ref="modal" title="Location Info" @ok="closeModal" >
        {{selectedLocation}}
      </b-modal>
    </div>
  </div>
</div>
</template>

<style>
.overlay {
    position: absolute;   
    top: 0;               
    right: 0;             
    bottom: 0;
    left: 0;
    opacity: 0.5;
    z-index: 3000;
}
</style>
<script>
  import Vue from 'vue'
  import * as VueGoogleMaps from 'vue2-google-maps'
  import axios from 'axios'
  import HourGlass from 'vue-loading-spinner/src/components/HourGlass.vue'
  import DatePicker from 'vue2-datepicker'
  Vue.use(VueGoogleMaps, {
    load: {
      key: 'AIzaSyBW6jkW1QIMpjw5SUIytKmuQ40emkWLgM8',
      v: '3'
    }
  })
  function getBadminInfoList () {
    return axios.get('https://qatbadmap.herokuapp.com/api/badmintoninfolist')
  }

  function getLocationInfoList () {
    return axios.get('https://qatbadmap.herokuapp.com/api/locationinfolist')
  }

  export default {
    data () {
      return {
        center: { lat: 25.079613, lng: 121.556082 },
        zoom: 15,
        markers: [],
        locationList: [],
        badmintonList: [],
        errorMsg: '',
        selected: '',
        matchLocation: [],
        loading: true,
        selectedLocation: {},
        startTime: '',
        endTime: '',
        todayWeekday: 0
      }
    },
    created () {
      var self = this
      self.getLocation()
      axios.all([getBadminInfoList(), getLocationInfoList()]).then(axios.spread(function (badmintonResponse, locationResponse) {
        self.badmintonList = badmintonResponse.data
        self.locationList = locationResponse.data
        self.loading = false
        self.updateMap()
      }))
      var today = new Date()
      self.todayWeekday = today.getDay()
    },
    watch: {
      selected: function (val) {
        this.markers = this.badmintonList.filter(function (badmintonInfo) {
          return val === '' || badmintonInfo.location === val
        }
        )
        this.center = this.markers[0].position
      }
    },
    methods: {
      updateMap () {
        var self = this
        this.badmintonList.forEach(function (badminton) {
          var matchLocation = self.locationList.filter(function (location) { return location.name === badminton.location })

          matchLocation ? badminton.position = {'lng': matchLocation[0].lng, 'lat': matchLocation[0].lat} : ''
          matchLocation ? badminton.infoText = badminton.location + ' Start Time: ' + badminton.startTime : ''
        })
        this.markers = this.badmintonList
        // this.center = this.markers[0].position
      },
      closeModal (e) {
        return e.cancel()
      },
      showModal (marker) {
        this.selectedLocation = marker
        this.$refs.modal.show()
      },
      showTodayInfo () {
        var self = this
        this.markers = this.badmintonList.filter(function (location) { return location.weekDayInt === self.todayWeekday })
        this.center = this.markers[0].position
      },
      getLocation () {
        var self = this
        if (navigator.geolocation) {
          return navigator.geolocation.getCurrentPosition(function (position) { self.center = {lat: position.coords.latitude, lng: position.coords.longitude} })
        } else {
          console.log('Geolocation is not supported by this browser.')
          return { lat: 25.079613, lng: 121.556082 }
        }
      }
    },
    components: {
      HourGlass,
      DatePicker
    }
  }
</script>
