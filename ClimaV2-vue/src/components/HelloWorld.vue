<script setup>
import { computed, ref } from 'vue'
import BuscarCiudad from './BuscarCiudad.vue'
import DetallesMet from './DetallesMet.vue'
import InfoClima from './InfoClima.vue'

// El input guarda la ciudad escrita y solicitud activa la búsqueda del geocodificador.
const text = ref('Puebla')
const solicitud = ref({ query: '' })

// Estos estados reciben la ciudad, el clima y errores de cada etapa del proceso.
const resultado = ref(null)
const clima = ref(null)
const error = ref('')
const errorClima = ref('')

// Obtiene el mismo periodo de seis horas que usa DetallesHora para elegir el fondo.
const periodoDelDia = computed(() => {
  const horaTexto = clima.value?.current?.time
  const hora = Number(horaTexto?.slice(11, 13))

  if (!Number.isInteger(hora) || hora < 0 || hora > 23) return null
  if (hora >= 0 && hora < 6) return 1
  if (hora >= 6 && hora < 12) return 2
  if (hora >= 12 && hora < 18) return 3
  if (hora >= 18 && hora < 24) return 4

  return null
})

// Asigna una clase CSS que cambia el color de fondo para cada periodo horario.
const claseFondo = computed(() => (
  periodoDelDia.value ? `fondo-periodo-${periodoDelDia.value}` : 'fondo-periodo-inicial'
))

// Valida la entrada y prepara una nueva solicitud para BuscarCiudad.
function buscarCiudad() {
  // Limpia resultados y errores previos, y valida el texto antes de buscar.
  const query = text.value.trim()
  resultado.value = null
  clima.value = null
  error.value = ''
  errorClima.value = ''

  if (!query) {
    // INPUT_EMPTY: puede ocurrir si se pulsa Buscar con el campo vacío o con espacios.
    error.value = {
      code: 'INPUT_EMPTY',
      message: 'Escribe el nombre de una ciudad.'
    }
    return
  }

  solicitud.value = { query }
}

// Recibe la ciudad y sus coordenadas; estas activan la consulta meteorológica.
function mostrarResultado(ciudad) {
  resultado.value = ciudad
  clima.value = null
  errorClima.value = ''
}

// Muestra en la interfaz los errores estructurados reportados por Nominatim.
function mostrarError(mensaje) {
  error.value = mensaje
}

// Guarda el clima de Open-Meteo; el cambio también actualiza fondo y detalles.
function mostrarClima(datos) {
  clima.value = datos
}

// Mantiene separados los errores de geocodificación y los de Open-Meteo.
function mostrarErrorClima(mensaje) {
  errorClima.value = mensaje
}

</script>

<template>
  <!-- La clase cambia el fondo de toda la página según la hora meteorológica local. -->
  <div class="pagina" :class="claseFondo">
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
  <p v-if="error" role="alert">Error {{ error.code }}: {{ error.message }}</p>
  <p v-if="errorClima" role="alert">
    Error {{ errorClima.code }}: {{ errorClima.message }}
  </p>
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
  <section id="spacer"><br><p>Desarrollado como Actividad 8 para la materia de Redes y Servicios</p></section>
  </div>
  
</template>
