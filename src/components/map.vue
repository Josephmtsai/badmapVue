<template>
  <div  class="text-center">
    <b-btn variant="success"  ref="btn_click"   @click="getLocation">Get Location  </b-btn>
    <b-dropdown text="Right align" variant="warning" right class="m-md-2">
        <b-dropdown-item href="#" v-for="location in locationList" v-bind:item="item"   v-bind:index="location"  v-bind:key="location.name" >{{location.name}}</b-dropdown-item>
    </b-dropdown>

    <b-form-select v-model="selected"   value-field="name" text-field="name"  :options="locationList"  class="mb-3" ></b-form-select>
    <gmap-map   :center="center"  :zoom="zoom"  map-type-id="terrain"  style="width: 1000px; height: 300px">
          <gmap-marker :key="index" v-for="(m, index) in markers" :position="m.position" :clickable="true" @click="center=m.position" ></gmap-marker>
    </gmap-map>

    {{selected}}
</div>

</template>
<script>
  import Vue from 'vue'
  import * as VueGoogleMaps from 'vue2-google-maps'
  Vue.use(VueGoogleMaps, {
    load: {
      key: 'AIzaSyBW6jkW1QIMpjw5SUIytKmuQ40emkWLgM8',
      v: '3'
    }
  })

  export default {
    data () {
      return {
        center: { lat: 25.079613, lng: 121.556082 },
        zoom: 13,
        markers: [],
        locationList: [],
        errorMsg: '',
        selected: '',
        matchLocation: []
      }
    },
    watch: {
      selected: function (val) {
        this.matchLocation = this.locationList.filter(function (location) {
          return location.name === val
        }
        )
        this.markers = this.matchLocation.map(({ lng, lat, name }) => {
          return {
            position: { lng, lat },
            infoText: name
          }
        })
      }
    },
    methods: {
      getLocation () {
        const api = 'https://qatbadmap.herokuapp.com/api/locationinfolist'
        Vue.axios.get(api).then(response => {
          this.locationList = response.data
        })
      },
      updateMap () {
        this.markers = this.locationList.filter(function (location) {
          return location.name === this.selected
        }
        )
        alert(this.markers)
      }
    }
  }
</script>
