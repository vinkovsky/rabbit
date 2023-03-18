<template>
  <Bunny msg="Welcome to Your Vue.js App" />
</template>

<script>
import Bunny from "./components/Bunny.vue";

export default {
  name: "App",
  components: {
    Bunny,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
html,
body {
  width: 100%;
  height: 100%;
  font-family: "Courier New", Courier, monospace;
  font-size: 95%;
  margin: 0; 
  height: 100%; 
  overflow: hidden

}
</style>
