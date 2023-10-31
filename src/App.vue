<template>
  <div id="app">
    <CanvasDraw />
  </div>
</template>

<script>
import CanvasDraw from "./components/CanvasDraw.vue";

export default {
  name: "App",
  components: {
    CanvasDraw,
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
</style>
