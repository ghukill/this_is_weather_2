<template>
  <div id="app">
    <main>
        <!-- pass props here... -->
        <router-view></router-view>
    </main>
  </div>
</template>

<script>
  // import Weather from './components/Weather';
  export default {
    name: 'app',
    // data () {
    //   return {
    //     search_term: ''
    //   }
    // },
    // methods: {
    //   use_current_location: function() {}
    // },
    // components: {
    //   appWeather: Weather
    // }
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


