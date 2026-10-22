<script setup>
import { ref, watch } from 'vue'

// Recibe la ciudad encontrada y comunica al componente padre el clima obtenido.
const props = defineProps({
  lugar: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['resultado', 'error'])
const buscando = ref(false)

// Mantiene el formato de error consistente para HelloWorld.
function emitirError(code, message) {
  emit('error', { code, message })
}

// Consulta al recibir las coordenadas y cuando cambian; immediate cubre el montaje
// con una ciudad ya seleccionada.
watch(() => props.lugar, async (lugar, _, onCleanup) => {
  if (!lugar) return

  buscando.value = false

  // WEATHER_INVALID_COORDINATES: protege la API de coordenadas ausentes o no numéricas.
  if (!Number.isFinite(lugar.latitud) || !Number.isFinite(lugar.longitud)) {
    emitirError('WEATHER_INVALID_COORDINATES', 'La ciudad no contiene coordenadas válidas.')
    return
  }

  // Cancela la consulta anterior si se inicia otra búsqueda antes de terminar.
  const controller = new AbortController()
  onCleanup(() => controller.abort())
  buscando.value = true

  try {
    // Open-Meteo recibe las coordenadas de Nominatim y devuelve las condiciones actuales.
    const apiUrl = new URL('https://api.open-meteo.com/v1/forecast')
    apiUrl.search = new URLSearchParams({
      latitude: String(lugar.latitud),
      longitude: String(lugar.longitud),
      current: 'relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m,temperature_2m',
      timezone: 'auto'
    })

    const respuesta = await fetch(apiUrl, { signal: controller.signal })
    // WEATHER_HTTP_ERROR: puede deberse a parámetros inválidos, límite o indisponibilidad.
    if (!respuesta.ok) {
      emitirError(
        'WEATHER_HTTP_ERROR',
        `Open-Meteo respondió con HTTP ${respuesta.status}.`
      )
      return
    }

    const clima = await respuesta.json()
    const actual = clima.current

    // WEATHER_INVALID_RESPONSE: la API no incluyó uno o más valores esperados.
    if (
      !actual ||
      !Number.isFinite(actual.temperature_2m) ||
      !Number.isFinite(actual.apparent_temperature) ||
      !Number.isFinite(actual.relative_humidity_2m) ||
      !Number.isFinite(actual.weather_code) ||
      !Number.isFinite(actual.wind_speed_10m)
    ) {
      emitirError(
        'WEATHER_INVALID_RESPONSE',
        'Open-Meteo devolvió datos meteorológicos incompletos o con formato inesperado.'
      )
      return
    }

    // Devuelve al padre la respuesta completa, incluidas unidades, hora y zona horaria.
    emit('resultado', clima)
  } catch (error) {
    // Una cancelación ocurre al cambiar de ciudad; no debe mostrarse como fallo.
    if (error instanceof Error && error.name === 'AbortError') return

    // WEATHER_INVALID_RESPONSE: JSON inválido; WEATHER_NETWORK_ERROR: red, DNS o CORS.
    if (error instanceof SyntaxError) {
      emitirError('WEATHER_INVALID_RESPONSE', 'Open-Meteo devolvió una respuesta que no es JSON válido.')
    } else if (error instanceof TypeError) {
      emitirError(
        'WEATHER_NETWORK_ERROR',
        'No se pudo conectar con Open-Meteo; revisa la conexión, DNS o las restricciones CORS.'
      )
    } else {
      emitirError('WEATHER_UNEXPECTED_ERROR', 'Ocurrió un error inesperado al consultar el clima.')
    }
  } finally {
    if (!controller.signal.aborted) buscando.value = false
  }
}, { immediate: true })
</script>

<template>
  <!-- Informa al usuario mientras se consulta la API meteorológica. -->
  <p v-if="buscando" role="status">Consultando datos meteorológicos...</p>
</template>
