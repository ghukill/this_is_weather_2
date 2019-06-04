<template>
  <div id="app">
    <main>
        <!-- pass props here... -->
        <router-view></router-view>
    </main>
  </div>
</template>

<script>
  export default {
    name: 'app'
  }
</script>

<style>
  body {
    padding-top: 4.5rem;
    /*background-color: black;*/
    /*color:white;*/
  }
  .container {
    max-width:95%;
  }
  #render_conditions td.reading {
    font-family:courier;
  }
</style>


