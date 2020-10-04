<template>
  <div class="root">
    <Minesweeper></Minesweeper>
  </div>
</template>

<script>
import Minesweeper from "./components/Minesweeper.vue";

export default {
  name: 'App',
  components: {
    Minesweeper
  }
}
</script>

<style>
body, html{
  padding: 0;
  margin: 0;
}
</style>
