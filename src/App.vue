<template>
  <div id="app">
      <div class="logo">
          <img src="./assets/logo.png" alt="vue" class="img-logo">
      </div>
      <h1>Buscador de países</h1>
      <p class="instrucciones">Es recomendable hacer las búsquedas en ingles.</p>
      <buscador></buscador>
  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'
import Buscador from './components/Buscador';

export default {
  name: 'App',
  components: {
    //HelloWorld
    Buscador
  }
}
</script>

<style>
#app {
  font-family: "Lato", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #44476A;
  margin-top: 60px;
}

.logo{
    background-color: #ecececec;
    background-size: 115px !important;
    background-position: center !important;
    background-repeat: no-repeat;
    width: 180px;
    height: 180px;
    border-radius: 50%;
    margin: 0 auto;
    box-shadow: /* logo shadow */ 0px 0px 2px #5f5f5f,
                /* offset */ 0px 0px 0px 5px #ecf0f3,
                /* Bottom rigth */ 8px 8px 15px #a7aaaf,
                /* Top Left */ -8px -8px 15px #ffffff;
    background-size: cover;
    background-position: inherit;
}

.img-logo {
    margin-top: 32px;
    width: 150px;
    height: 150px;
}

h1 {
    margin-bottom: 0px;
}

.instrucciones {
    font-size: 0.9rem;
    margin-top: 0px;
}

</style>
