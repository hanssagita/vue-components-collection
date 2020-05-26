<template>
  <div id="app">
    <CatCarouselDemo />
  </div>
</template>

<script>
import CatCarouselDemo from '@/demo/CatCarouselDemo'
export default {
  name: 'App',
  components: {
    CatCarouselDemo
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
