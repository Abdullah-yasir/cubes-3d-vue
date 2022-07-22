<template>
  <div id="app">
    <VolexPainter />
  </div>
</template>

<script>
import VolexPainter from "./components/VolexPainter.vue";

export default {
  name: "App",
  components: {
    VolexPainter,
  },
};
</script>

<style>
body {
  margin: 0px;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
