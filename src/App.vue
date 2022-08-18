<template>
  <v-app>
    <!--<v-app-bar
      app
      color="primary"
      dark
    >

    </v-app-bar>-->

    <v-main>
      <video src="./assets/vid/flag.mp4" autoplay loop></video>
      <Candidates/>
    </v-main>
  </v-app>
</template>

<script>
import Candidates from './components/Candidates.vue';
//import HelloWorld from './components/HelloWorld';

export default {
  name: 'App',
  components: {
    Candidates,
  },

  data: () => ({
    //
  }),
};
</script>

<style>
video{
    width: 100vw;
    position: absolute;
    top:0px;
    left: 0px;
    z-index: 1;
}
body{
    overflow-y: hidden;
}
</style>
