<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <div id="app">
    <CountdownImage />
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import CountdownImage from './components/CountdownImage.vue';
export default {
  // name: 'App',
  name: 'App',
  components: {
    // HelloWorld,
    CountdownImage,
  }
}
</script>


<style>
/* #app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
} */

#app {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
</style>
