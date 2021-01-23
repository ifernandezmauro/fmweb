<template>
  <div id="app">
    <graph-wrap/>
  </div>
</template>

<script>
import GraphWrap from "./components/GraphWrap";

export default {
  name: 'App',
  components: {
    GraphWrap
  }
}
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
