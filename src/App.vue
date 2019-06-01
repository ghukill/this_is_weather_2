<template>
  <div id="app">
    <main>
      <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
        <a class="navbar-brand" href="#">THIS IS WEATHER 2</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse" aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item active">
              <a class="nav-link" href="#">Bird's Eye</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">Precipitation</a>
            </li>
          </ul>
          <form id="location_input" class="form-inline mt-2 mt-md-0" v-on:submit.prevent="search">
            <input class="form-control mr-sm-2" type="text" v-model="search_term" placeholder="search location...">
            <button class="btn btn-outline-success my-2 my-sm-0" type="submit">Search</button>
          </form>
          <button v-on:click="use_current_location" style="margin-left:7px;" class="btn btn-success my-2 my-sm-0">Current Location</button>
        </div>
      </nav>
      <div role="main" class="container">
        <router-view :search_term="search_term"></router-view>
      </div>
    </main>
  </div>
</template>

<script>
  import Weather from './components/Weather';
  export default {
    name: 'app',
    data () {
      return {
        search_term: ''
      }
    },
    methods: {
      use_current_location: function() {}
    },
    components: {
      appWeather: Weather
    }
  }
</script>

<style>
  body {
    padding-top: 4.5rem;
  }
  .container {
    max-width:95%;
  }
</style>


