<template>

  <div class="weather">

    <div class="row">
      <div class="col-md-8">
        <h3 v-if="selected_osm_location != undefined"><code>{{ selected_osm_location.display_name }} ({{ lat }}, {{ lon }})</code></h3>
      </div>
      <div class="col-md-4">
        <h3 v-if="search_term != ''">Search term: <code>{{ search_term }}</code></h3>
        <template v-if="osm_location_disambiguate && search_term != ''">
          <ul>
            <li v-for="location in osm_locations">
              <a v-on:click.prevent="select_location(location)" href="">{{ location.display_name }}</a>
            </li>
          </ul>
        </template>
      </div>
    </div>

    <!-- show conditions -->
    <div id="render_conditions">

      <!-- conditions table -->
      <div class="row">
        <div class="col-md-6">
          <table class="table">
            <tr>
              <th>Temperature</th>
              <td></td>
            </tr>
            <tr>
              <th>Humidity</th>
              <td></td>
            </tr>
          </table>
        </div>
        <div class="col-md-6">
          <table class="table">
            <tr>
              <th>Pressure</th>
              <td></td>
            </tr>
            <tr>
              <th>Something else</th>
              <td></td>
            </tr>
          </table>
        </div>
      </div>

      <!-- rainviewer -->
      <div class="row">
        <div class="col-md-12">
          <iframe id="rainviewer_iframe" v-bind:src="'https://www.rainviewer.com/map.html?loc=' + lat + ',' + lon + ',7&oFa=0&oC=0&oU=0&oCS=0&oF=1&oAP=0&rmt=3&c=5&o=70&lm=0&th=1&sm=0&sn=1'" width="100%" frameborder="0" style="border:0;height:50vh;" allowfullscreen></iframe>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
  export default {
    name: 'goober',
    props: ['search_term'],
    data () {
      return {
        osm_locations: [],
        osm_location_disambiguate: false,
        selected_osm_location: null,
        lat: undefined,
        lon: undefined,
      }
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

        // turn on disambiguation window
        this.osm_location_disambiguate = true;

        // init OSM client and fire OSM nomination search
        let osm_client = new OSMNominationClient(this);
        osm_client.search_location(this.search_term)
          .then(function(data){
            vm.$data.osm_locations = data;
          })

      },

      select_location: function (location){

        // set selected location
        this.selected_osm_location = location;

        // set latitude and longitude
        this.lat = location.lat;
        this.lon = location.lon;

        // get conditions for this area
        this.get_conditions();

      },

      get_conditions: function () {

        // query darksky.net api
        // https://api.darksky.net/forecast/ab39481591505f01b53e976e1081f4d6/42.2410562,-83.613055

      }
    }
  }
</script>


