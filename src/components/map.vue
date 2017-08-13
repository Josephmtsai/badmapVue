<template>
  <div  class="text-center col-12">
    <div align="center" v-if="loading" class="col-12 overlay">
      <hour-glass size="500px"></hour-glass>
    </div>
    <div class="col-12" v-if="!loading">
      <b-card no-block>
        <b-tabs small card ref="tabs" v-model="tabIndex">
          <b-tab id="mapTab" title="General Map">

            <b-form-fieldset label="每周選項"  label-for="weeklist" >
              <div role="group" id="checkboxes2">
                <b-form-checkbox  v-for="weekDayValue in weekDaysOptions" v-model="weekDays"  :value="weekDayValue.value">{{weekDayValue.label}}</b-form-checkbox>
              </div>
            </b-form-fieldset>
            <gmap-map  class="col-12" style="height:800px" :center="center"  :zoom="zoom"  map-type-id="roadmap"  >
              <gmap-cluster >
                  <gmap-marker :key="index" v-for="m in markers" :position="m.position" :clickable="true" @click="showModal(m)" :label="m.name"  ></gmap-marker>
              </gmap-cluster>
            </gmap-map>
          </b-tab>
          <b-tab id="listTab" title="List Location Info">
            <span class="label label-default" for="selectLocation">Specify Location</span>
            <div >
              <b-form-select v-model="selected"  id="selectLocation" value-field="name" text-field="name"  :options="locationList"  class="mb-3" ></b-form-select>
            </div>
          </b-tab>
        </b-tabs>
      </b-card>
      <b-modal ref="modal" title="Location" @ok="closeModal" size="lg" >
        <b-table striped hover show-empty :items="selectedBadmintonInfo"  :fields="badmintonFields" :sort-by.sync="sortBy" :sort-desc.sync="sortDesc"  >
          <template slot="location" scope="row">{{row.value}}</template>
          <template slot="startTime" scope="row">{{row.value}}</template>
          <template slot="endTime" scope="row">{{row.value}}</template>
          <template slot="payInfo" scope="row">{{row.value}}</template>
          <template slot="contactName" scope="row">{{row.value}}</template>
          <template slot="contactPhone" scope="row">{{row.value}}</template>
          <template slot="line" scope="row">{{row.value}}</template>
        </b-table>
      </b-modal>
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
        tabIndex: 0,
        markers: [],
        locationList: [],
        badmintonList: [],
        errorMsg: '',
        selected: '',
        matchLocation: [],
        loading: true,
        selectedBadmintonInfo: [],
        filterBadmintonList: [],
        sortBy: null,
        sortDesc: false,
        badmintonFields: {
          location: { label: '地點', sortable: true },
          startTime: { label: '開始時間', sortable: true },
          endTime: { label: '結束時間', sortable: true },
          payInfo: { label: '金額', sortable: true, 'class': 'text-center' },
          contactName: { label: '聯絡人' },
          contactPhone: { label: '電話', sortable: true, 'class': 'text-center' },
          line: { label: 'Line', sortable: true, 'class': 'text-center' }
        },
        dateTimeRange: [],
        todayWeekday: 0,
        weekDays: [],
        weekDaysOptions: [{ 'label': '星期日', 'value': 0 }, { 'label': '星期一', 'value': 1 }, { 'label': '星期二', 'value': 2 }, { 'label': '星期三', 'value': 3 }, { 'label': '星期四', 'value': 4 }, { 'label': '星期五', 'value': 5 }, { 'label': '星期六', 'value': 6 }]
      }
    },
    created () {
      var self = this
      var today = new Date()
      self.todayWeekday = today.getDay()
      // default 3 days
      self.weekDays.push(self.todayWeekday)
      self.weekDays.push(self.todayWeekday + 1 > 6 ? 0 : self.todayWeekday + 1)
      // self.weekDays.push(self.todayWeekday + 2 > 6 ? 1 : self.todayWeekday + 2)
      axios.all([getBadminInfoList(), getLocationInfoList()]).then(axios.spread(function (badmintonResponse, locationResponse) {
        self.badmintonList = badmintonResponse.data
        self.locationList = locationResponse.data
        self.loading = false
        self.updateMap()
      }))
    },
    watch: {
      selected: function (val) {
        this.markers = this.badmintonList.filter(function (badmintonInfo) {
          return val === '' || badmintonInfo.location === val
        }
        )
        this.center = this.markers[0].position
      },
      weekDays: function (val) {
        this.updateMap()
      }
    },
    methods: {
      updateMap () {
        var self = this
        this.filterBadmintonList = this.badmintonList.filter(function (badminton) {
          return self.weekDays.indexOf(badminton.weekDayInt) !== -1
        })
        var filterLocationList = this.locationList.filter(function (location) {
          return self.filterBadmintonList.filter(function (badminton) {
            return location.name === badminton.location
          }).length > 0
        })
        this.markers = filterLocationList
        self.getLocation()
      },
      closeModal (e) {
        return e.cancel()
      },
      showModal (marker) {
        this.selectedBadmintonInfo = this.filterBadmintonList.filter(function (badminton) {
          return marker.name === badminton.location
        })
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
          return navigator.geolocation.getCurrentPosition(function (position) { self.center = {'lat': position.coords.latitude, 'lng': position.coords.longitude} })
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
