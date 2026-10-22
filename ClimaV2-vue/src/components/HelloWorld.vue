<script setup>
import { ref } from 'vue'
import heroImg from '../assets/hero.png'
import viteLogo from '../assets/vite.svg'
import vueLogo from '../assets/vue.svg'

const count = ref(0)
const text = ref('Puebla')


</script>

<template>
  <section id="center">
    <div>
      <h1>Clima Pro Version 2!</h1>
      <p>Ahora con <code>Vue y Vite</code> como <code>Framework Frontend Moderno</code></p>
    </div>
    <h2>Escribe el nombre de alguna ciudad aqui:</h2>
  <section style="display: flex; flex-direction: row;">
  <input v-model="text" class="counter" id="CiudadUsuario">
  <p style="margin: 10px;"></p>
  <button class="counter">Buscar</button>
  </section>
  <p id="Ciudad"></p>
  
  </section>

  <div class="ticks"></div>

  <section id="next-steps">
    <div id="docs">
            <h1 id="ciudad">Ciudad: </h1>
            <h1 id="error"></h1>
            <h2 id="temperatura">Temperatura: </h2>
            <h2 id="STermica">Sensación térmica:</h2>
            <h2 id="humedad">Humedad:</h2>
            <h2 id="CMeteorologica">Condición metereológica:</h2>
            <h2 id="VViento">Velocidad del viento:</h2>
            <h3 id="longitud">Longitud:</h3>
            <h3 id="latitud">Latitud:</h3>
    </div>
    <div id="social">
      <h1 id="Hora:">Ciudad: </h1>
      <h1 id="error"></h1>
        <h2 id="zona-horaria">Zona Horaria: </h2>
        <h1 id="emoji">😎</h1>
    </div>
  </section>

  <div class="ticks"></div>
  <section id="spacer"></section>
</template>
