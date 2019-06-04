<template>

  <div class="weather">

    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <a class="navbar-brand" href="#">THIS IS WEATHER 2</a>
      <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse" aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarCollapse">
        <ul class="navbar-nav mr-auto">
<!--          <li class="nav-item active">-->
<!--            <a class="nav-link" href="#">Bird's Eye</a>-->
<!--          </li>-->
<!--          <li class="nav-item">-->
<!--            <a class="nav-link" href="#">Precipitation</a>-->
<!--          </li>-->
        </ul>
        <form id="location_input" class="form-inline mt-2 mt-md-0" v-on:submit.prevent="search">
          <input class="form-control mr-sm-2" type="text" v-model="search_term" placeholder="search location...">
          <button class="btn btn-outline-success my-2 my-sm-0" type="submit">Search</button>
        </form>
        <button v-on:click="use_current_location" style="margin-left:7px;" class="btn btn-success my-2 my-sm-0">Current Location</button>
      </div>
    </nav>

    <div role="main" class="container">

      <div class="row">
        <div class="col-md-12">
          <template v-if="osm_location_disambiguate">
            <h3 v-if="search_term != ''">Search term: <code>{{ search_term }}</code></h3>
            <ul>
              <li v-for="location in osm_locations">
                <a v-on:click.prevent="select_location(location)" href="">{{ location.display_name }}</a>
              </li>
            </ul>
          </template>
        </div>
      </div>

      <div class="row">
        <div class="col-md-8">
          <h3 v-if="selected_osm_location != undefined">{{ selected_osm_location.display_name }} ({{ lat }}, {{ lon }})</h3>
        </div>

      </div>

      <!-- show conditions -->
      <div id="render_conditions" v-if="selected_osm_location != null && observations != undefined">

        <div class="row">
          <div class="col-md-6">
            <table class="table">
              <tr>
                <th>Temperature</th>
                <td class="reading">{{observations.currently.apparentTemperature}}&#176; F</td>
              </tr>
              <tr>
                <th>Humidity</th>
                <td class="reading">{{observations.currently.humidity * 100}}%</td>
              </tr>
            </table>
          </div>
          <div class="col-md-6">
            <table class="table">
              <tr>
                <th>Pressure</th>
                <td class="reading">{{observations.currently.pressure}} mbar</td>
              </tr>
              <tr>
                <th>wind Gusts</th>
                <td class="reading">{{observations.currently.windGust}} mph</td>
              </tr>
            </table>
          </div>
        </div>

        <!-- rainviewer -->
        <div class="row">
          <div class="col-md-12">
            <iframe id="rainviewer_iframe" v-bind:src="'https://www.rainviewer.com/map.html?loc=' + lat + ',' + lon + ',7&oFa=0&oC=0&oU=0&oCS=0&oF=1&oAP=0&rmt=3&c=5&o=50&lm=0&th=1&sm=0&sn=1'" width="100%" frameborder="0" style="border:0;height:50vh;" allowfullscreen></iframe>
          </div>
        </div>

        <div class="row">
          <div class="col-md-12">
            <line-chart :line_data="line_data"></line-chart>
          </div>
        </div>

      </div>
    </div>

  </div>



</template>

<script>

  // function to convert epoch to hour
  function hour_from_epoch(t) {
    let d = new Date(t);
    return d.getHours();
  }

  // import components
  import LineChart from './LineChart.vue'

  export default {
    name: 'weather',
    data () {
      return {
        search_term: '',
        osm_locations: [],
        osm_location_disambiguate: false,
        selected_osm_location: null,
        lat: undefined,
        lon: undefined,
        observations: undefined
      }
    },
    computed:{
      line_data: function() {
        let line_data = this.observations.hourly.data.map(x => ({label:hour_from_epoch(x.time), value:x.apparentTemperature}) );
        return line_data;
      }
    },
    components:{
      'line-chart':LineChart
    },
    methods: {
      use_current_location: function() {

        // get position
        var getPosition = function (options) {
          return new Promise(function (resolve, reject) {
            navigator.geolocation.getCurrentPosition(resolve, reject, options);
          });
        }

        getPosition()
          .then((position) => {

            // set as null
            vm.$data.selected_osm_location = {
              display_name: "Current Location"
            };

            // set coords
            vm.$data.lat = position.coords.latitude;
            vm.$data.lon = position.coords.longitude;

            // update conditions
            vm.$root.get_conditions();

          })
          .catch((err) => {
            console.error(err.message);
          });
      },

      search: function (){

        // set self for then context
        var self = this;

        // turn on disambiguation window
        this.osm_location_disambiguate = true;

        // init OSM client and fire OSM nomination search
        let osm_client = new OSMNominationClient(this);
        osm_client.search_location(this.search_term)
          .then(function(data){
            self.osm_locations = data;
          })

      },

      select_location: function (location){

        // set disambiguation flag to false
        this.osm_location_disambiguate = false;

        // set selected location
        this.selected_osm_location = location;

        // set latitude and longitude
        this.lat = location.lat;
        this.lon = location.lon;

        // get conditions for this area
        this.get_conditions();

      },

      get_conditions: function () {

        // set self for then context
        var self = this;

        // query darksky.net api
        // https://api.darksky.net/forecast/ab39481591505f01b53e976e1081f4d6/42.2410562,-83.613055
        // temporary hack, using this proxy for DarkSky API
        // https://csm.fusioncharts.com/files/assets/wb/wb-data.php?src=darksky&lat=40.7128&long=83.613

        $.get(`https://csm.fusioncharts.com/files/assets/wb/wb-data.php?src=darksky&lat=${this.lat}&long=${this.lon}`)
          .then(function(data){
            self.observations = JSON.parse(data);
          });

      }
    }
  }



  /**
   * Client for OpenStreetMaps (OSM) Nomination tool
   *
   * API spec: https://nominatim.org/release-docs/develop/
   */
  class OSMNominationClient {

    // API base url
    base_url = 'https://nominatim.openstreetmap.org/search/'

    // search_location
    search_location(search_term) {

      return $.get( `${this.base_url}/${search_term}?format=json`).then(function(data){
        return data;
      });

    }

  }



</script>


