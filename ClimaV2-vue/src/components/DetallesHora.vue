<script setup>
import { computed } from 'vue'

// Recibe la hora local y la zona horaria incluidas en la respuesta de Open-Meteo.
const props = defineProps({
  clima: {
    type: Object,
    default: null
  }
})

const emojis = {
  1: '🌇',
  2: '🏙️',
  3: '🌆',
  4: '🌃'
}

// Clasifica la hora en cuatro periodos consecutivos de seis horas.
const TIEMPO = computed(() => {
  const horaObservacion = props.clima?.current?.time
  const hora = Number(horaObservacion?.slice(11, 13))

  // Si la API todavía no ha entregado una hora válida, no asigna un periodo.
  if (!Number.isInteger(hora) || hora < 0 || hora > 23) return null

  if (hora >= 0 && hora < 6) return 1
  if (hora >= 6 && hora < 12) return 2
  if (hora >= 12 && hora < 18) return 3
  if (hora >= 18 && hora < 24) return 4

  return null
})
</script>

<template>
    <div id="social">
      <!-- Muestra la hora de Open-Meteo, el periodo de seis horas y su emoji. -->
      <h1 style="line-height: normal;">Hora de observación: {{ clima?.current?.time || '—' }}</h1>
      <h2>Zona horaria: {{ clima?.timezone || '—' }}</h2>
      <h1 id="emoji">{{ emojis[TIEMPO] || '😎' }}</h1>
    </div>
</template>
