<script setup>
import { ref, watch } from 'vue'

// Recibe la búsqueda desde HelloWorld y devuelve el resultado mediante eventos.
const props = defineProps({
  solicitud: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['resultado', 'error'])
const buscando = ref(false)

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

    // Convierte errores HTTP y respuestas sin coincidencias en mensajes visibles.
    if (!respuesta.ok) {
      throw new Error(`No fue posible buscar la ciudad (HTTP ${respuesta.status}).`)
    }

    const lugares = await respuesta.json()
    if (!Array.isArray(lugares) || lugares.length === 0) {
      throw new Error('Ciudad no encontrada.')
    }

    const lugar = lugares[0]
    const latitud = Number(lugar.lat)
    const longitud = Number(lugar.lon)
    const nombre = lugar.name || lugar.display_name

    // Comprueba que la API haya devuelto los tres datos que necesita la vista.
    if (!nombre || !Number.isFinite(latitud) || !Number.isFinite(longitud)) {
      throw new Error('La API devolvió datos incompletos para esta ciudad.')
    }

    // Envía al padre los datos listos para mostrarse.
    emit('resultado', { nombre, latitud, longitud })
  } catch (error) {
    // Las peticiones canceladas son esperadas; otros errores se muestran al usuario.
    if (error instanceof Error && error.name === 'AbortError') return
    emit('error', error instanceof Error ? error.message : 'No se pudo buscar la ciudad.')
  } finally {
    if (!controller.signal.aborted) buscando.value = false
  }
})
</script>

<template>
  <span v-if="buscando" role="status">Buscando ciudad...</span>
</template>
