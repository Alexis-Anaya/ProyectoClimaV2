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

// Consulta al recibir las coordenadas y cuando cambian; immediate cubre el montaje
// con una ciudad ya seleccionada.
watch(() => props.lugar, async (lugar, _, onCleanup) => {
  if (!lugar) return

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
    if (!respuesta.ok) {
      throw new Error(`No fue posible consultar el clima (HTTP ${respuesta.status}).`)
    }

    const clima = await respuesta.json()
    const actual = clima.current

    // Verifica que la respuesta incluya todos los valores meteorológicos solicitados.
    if (
      !actual ||
      !Number.isFinite(actual.temperature_2m) ||
      !Number.isFinite(actual.apparent_temperature) ||
      !Number.isFinite(actual.relative_humidity_2m) ||
      !Number.isFinite(actual.weather_code) ||
      !Number.isFinite(actual.wind_speed_10m)
    ) {
      throw new Error('El servicio meteorológico devolvió datos incompletos.')
    }

    // Devuelve al padre la respuesta completa, incluidas unidades, hora y zona horaria.
    emit('resultado', clima)
  } catch (error) {
    // Ignora cancelaciones intencionales y comunica otros problemas a la interfaz.
    if (error instanceof Error && error.name === 'AbortError') return
    emit('error', error instanceof Error ? error.message : 'No se pudo consultar el clima.')
  } finally {
    if (!controller.signal.aborted) buscando.value = false
  }
}, { immediate: true })
</script>

<template>
  <!-- Informa al usuario mientras se consulta la API meteorológica. -->
  <p v-if="buscando" role="status">Consultando datos meteorológicos...</p>
</template>
