<template>
  <div id="app">
    <MapSnapshot />
  </div>
</template>

<script>
import MapSnapshot from "./components/MapSnapshot";

export default {
  name: "App",
  components: {
    MapSnapshot,
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>