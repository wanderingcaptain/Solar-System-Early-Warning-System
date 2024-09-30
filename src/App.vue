<template>
  <div id="app">
    <SolarSystem />
  </div>
</template>

<script>
import SolarSystem from './components/solarsystem.vue';

export default {
  name: 'App',
  components: {
    SolarSystem,
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
  margin-top: 60px;
}
</style>
