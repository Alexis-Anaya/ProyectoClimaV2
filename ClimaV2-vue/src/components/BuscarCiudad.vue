<script setup>
import { ref, watch } from 'vue'

// Recibe la búsqueda desde HelloWorld y devuelve datos o errores mediante eventos.
const props = defineProps({
  solicitud: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['resultado', 'error'])
const buscando = ref(false)

// Centraliza el formato de error para que la interfaz siempre reciba código y explicación.
function emitirError(code, message) {
  emit('error', { code, message })
}

// Consulta Nominatim cuando cambia la solicitud enviada por el componente padre.
watch(() => props.solicitud, async (solicitud, _, onCleanup) => {
  const ciudad = solicitud.query.trim()
  if (!ciudad) return

  // Cancela una petición anterior si el usuario inicia otra búsqueda.
  const controller = new AbortController()
  onCleanup(() => controller.abort())
  buscando.value = true

  try {
    // Prepara la búsqueda para obtener el primer resultado en español.
    const parametros = new URLSearchParams({
      q: ciudad,
      format: 'jsonv2',
      limit: '1',
      'accept-language': 'es'
    })
    const respuesta = await fetch(
      `https://nominatim.openstreetmap.org/search?${parametros}`,
      { signal: controller.signal }
    )

    // GEO_HTTP_ERROR: puede indicar límite de peticiones, bloqueo o fallo del servidor.
    if (!respuesta.ok) {
      emitirError(
        'GEO_HTTP_ERROR',
        `Nominatim respondió con HTTP ${respuesta.status}.`
      )
      return
    }

    const lugares = await respuesta.json()
    if (!Array.isArray(lugares)) {
      // GEO_INVALID_RESPONSE: la respuesta JSON no tiene el arreglo de lugares esperado.
      emitirError('GEO_INVALID_RESPONSE', 'Nominatim respondió con un formato de datos inesperado.')
      return
    }

    if (lugares.length === 0) {
      // GEO_NOT_FOUND: el nombre no coincide con un lugar reconocido por Nominatim.
      emitirError('GEO_NOT_FOUND', 'No se encontró una ciudad con ese nombre.')
      return
    }

    const lugar = lugares[0]
    const latitud = Number(lugar.lat)
    const longitud = Number(lugar.lon)
    const nombre = lugar.name || lugar.display_name

    // GEO_INVALID_RESPONSE: puede deberse a datos faltantes o a una respuesta inesperada.
    if (!nombre || !Number.isFinite(latitud) || !Number.isFinite(longitud)) {
      emitirError(
        'GEO_INVALID_RESPONSE',
        'Nominatim no devolvió el nombre o las coordenadas en el formato esperado.'
      )
      return
    }

    // Envía al padre los datos listos para mostrarse.
    emit('resultado', { nombre, latitud, longitud })
  } catch (error) {
    // Una cancelación ocurre normalmente al iniciar otra búsqueda; no es un error visible.
    if (error instanceof Error && error.name === 'AbortError') return

    // GEO_INVALID_RESPONSE: JSON inválido; GEO_NETWORK_ERROR: conexión, CORS o DNS.
    if (error instanceof SyntaxError) {
      emitirError('GEO_INVALID_RESPONSE', 'Nominatim devolvió una respuesta que no es JSON válido.')
    } else if (error instanceof TypeError) {
      emitirError(
        'GEO_NETWORK_ERROR',
        'No se pudo conectar con Nominatim; revisa la conexión, DNS o las restricciones CORS.'
      )
    } else {
      emitirError('GEO_UNEXPECTED_ERROR', 'Ocurrió un error inesperado al buscar la ciudad.')
    }
  } finally {
    if (!controller.signal.aborted) buscando.value = false
  }
})
</script>

<template>
  <span v-if="buscando" role="status">Buscando ciudad...</span>
</template>
