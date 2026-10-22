<script setup>
import { ref } from 'vue'
import BuscarCiudad from './BuscarCiudad.vue'
import DetallesMet from './DetallesMet.vue'
import InfoClima from './InfoClima.vue'

const text = ref('Puebla')
// Estado que conecta la búsqueda de ciudad, la consulta meteorológica y la vista.
const solicitud = ref({ query: '' })
const resultado = ref(null)
const clima = ref(null)
const error = ref('')
const errorClima = ref('')

function buscarCiudad() {
  // Limpia resultados y errores previos, y valida el texto antes de buscar.
  const query = text.value.trim()
  resultado.value = null
  clima.value = null
  error.value = ''
  errorClima.value = ''

  if (!query) {
    error.value = 'Escribe el nombre de una ciudad.'
    return
  }

  solicitud.value = { query }
}

// Recibe del componente BuscarCiudad los datos encontrados por la API.
function mostrarResultado(ciudad) {
  resultado.value = ciudad
  clima.value = null
  errorClima.value = ''
}

// Muestra en la interfaz los errores reportados por el buscador.
function mostrarError(mensaje) {
  error.value = mensaje
}

// Guarda la respuesta meteorológica para entregársela al componente InfoClima.
function mostrarClima(datos) {
  clima.value = datos
}

// Mantiene separados los errores de geocodificación y los errores del clima.
function mostrarErrorClima(mensaje) {
  errorClima.value = mensaje
}

</script>

<template>
  <section id="center">
    <div>
      <h1>Clima Pro Version 2!</h1>
      <p>Ahora con <code>Vue y Vite</code> como <code>Framework Frontend Moderno</code></p>
    </div>
    <h2>Escribe el nombre de alguna ciudad aqui:</h2>
  <section style="display: flex; flex-direction: row;">
  <!-- El input mantiene el texto de búsqueda; se acepta el botón o Enter. -->
  <input v-model="text" class="counter" id="CiudadUsuario" @keyup.enter="buscarCiudad">
  <p style="margin: 10px;"></p>
  <button class="counter" type="button" @click="buscarCiudad">Buscar</button>
  </section>
  <BuscarCiudad
    :solicitud="solicitud"
    @resultado="mostrarResultado"
    @error="mostrarError"
  />
  <p v-if="error" role="alert">{{ error }}</p>
  <p v-if="errorClima" role="alert">{{ errorClima }}</p>
  <br>
  
  </section>

  <div class="ticks"></div>

  <!-- Pasa las coordenadas de la ciudad a DetallesMet para consultar Open-Meteo. -->
  <DetallesMet
    v-if="resultado"
    :lugar="resultado"
    @resultado="mostrarClima"
    @error="mostrarErrorClima"
  />

  <!-- Presenta los datos de la ciudad y el clima que devuelve DetallesMet. -->
  <InfoClima :lugar="resultado" :clima="clima" />

  <div class="ticks"></div>
  <section id="spacer"></section>
</template>
