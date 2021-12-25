<template>
  <div id="app">
    <Dashboard />
    <a href="https://cors-anywhere.herokuapp.com/corsdemo" target="blank">Si le graphique ne s'affiche pas, demandez l'accès à CORS anywhere</a>
  </div>
</template>

<script>
import Dashboard from "./components/Dashboard.vue";

export default {
  name: "App",
  components: {
    Dashboard,
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
