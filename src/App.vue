<template>



  <div id="app">
    <div v-if="!isMobile()">
        <Map/>
    </div>
    <div v-else>
    This app does not work on Mobile devices, try switching to a desktop device.
    </div>



      

  </div>
</template>

<script>
import Map from './components/Map.vue'

export default {
  name: 'App',
  components: {
    Map
  },
  methods: {
    isMobile() {
   if(/Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)) {
     return true
   } else {
     return false
   }
  },
}
}
</script>

<style>

    .header {
        grid-area: header;
        font-size: 15px;
        height: 5vh;
        font-family: 'Gill Sans','Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
    }

    .logout-button {
        
        border-radius: 0.6vh;
        font-size:14px;
        margin-left:85%;
        margin-bottom:10%;
        background:rgb(243, 243, 243);
        border: 1px solid #c0bfbf;
        color: rgb(255, 65, 36);
        padding:0.5vh;
        text-decoration: none;
        transition: all 0.1s ease-in-out;
    }
    .logout-button:hover {

        background: #ffffff;
    }

    body {
        background-color: #ffffff;
    }
    h1 {
        padding: 0;
        margin-top: 0;
    }
    #app {
        
        
        font-family: Roboto, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        
        color:rgb(7, 7, 7);
    }

    
    
</style>