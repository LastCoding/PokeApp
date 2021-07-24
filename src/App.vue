<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/images/pokemon-logo.png" width="250" height="250">
    <PokeForm></PokeForm>
  </div>
</template>

<script>
import PokeForm from './components/PokeForm.vue';

export default {
  name: 'App',
  components: {
    PokeForm
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
