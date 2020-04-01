<template>
  <div id="app">
    <!-- <tree /> -->
    <editor />

  </div>
</template>

<script>
// import tree from "./components/tree.vue";
import editor from "./components/editor.vue";

export default {
  name: "App",
  components: {
    // tree,
    editor
  }
};
</script>

<style>
* {
  margin: 0;
}

#app {
  font-family: "Roboto", "Helvetica", "Arial", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin: 0;
}
</style>
