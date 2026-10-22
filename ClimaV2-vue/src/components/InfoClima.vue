<script setup>
import { computed } from 'vue'

// Recibe del padre la ciudad seleccionada y la respuesta de Open-Meteo.
const props = defineProps({
  lugar: {
    type: Object,
    default: null
  },
  clima: {
    type: Object,
    default: null
  }
})

// Traduce los códigos meteorológicos WMO que devuelve Open-Meteo a descripciones.
const condiciones = {
  0: 'Despejado',
  1: 'Principalmente despejado',
  2: 'Parcialmente nublado',
  3: 'Cubierto',
  45: 'Niebla',
  48: 'Niebla con escarcha',
  51: 'Llovizna ligera',
  53: 'Llovizna moderada',
  55: 'Llovizna intensa',
  56: 'Llovizna helada ligera',
  57: 'Llovizna helada intensa',
  61: 'Lluvia ligera',
  63: 'Lluvia moderada',
  65: 'Lluvia intensa',
  66: 'Lluvia helada ligera',
  67: 'Lluvia helada intensa',
  71: 'Nevada ligera',
  73: 'Nevada moderada',
  75: 'Nevada intensa',
  77: 'Granos de nieve',
  80: 'Chubascos ligeros',
  81: 'Chubascos moderados',
  82: 'Chubascos intensos',
  85: 'Chubascos de nieve ligeros',
  86: 'Chubascos de nieve intensos',
  95: 'Tormenta',
  96: 'Tormenta con granizo ligero',
  99: 'Tormenta con granizo intenso'
}

// Calcula la descripción del tiempo cuando llega o cambia la respuesta meteorológica.
const descripcionClima = computed(() => {
  const codigo = props.clima?.current?.weather_code
  return condiciones[codigo] || (codigo !== undefined ? `Código meteorológico ${codigo}` : '')
})
</script>

<template>
  <section id="next-steps">
    <div id="docs">
      <!-- Presenta la ubicación y las condiciones actuales recibidas por props. -->
      <h1 id="ciudad" style="line-height: normal;">Ciudad: {{ lugar?.nombre || '' }}</h1>
      <h2 id="temperatura">
        Temperatura: {{ clima?.current?.temperature_2m ?? '—' }}{{ clima?.current_units?.temperature_2m || '' }}
      </h2>
      <h2 id="STermica">
        Sensación térmica: {{ clima?.current?.apparent_temperature ?? '—' }}{{ clima?.current_units?.apparent_temperature || '' }}
      </h2>
      <h2 id="humedad">
        Humedad: {{ clima?.current?.relative_humidity_2m ?? '—' }}{{ clima?.current_units?.relative_humidity_2m || '' }}
      </h2>
      <h2 id="CMeteorologica">Condición meteorológica: {{ descripcionClima }}</h2>
      <h2 id="VViento">
        Velocidad del viento: {{ clima?.current?.wind_speed_10m ?? '—' }}{{ clima?.current_units?.wind_speed_10m || '' }}
      </h2>
      <h3 id="longitud">Longitud: {{ lugar?.longitud ?? '' }}°</h3>
      <h3 id="latitud">Latitud: {{ lugar?.latitud ?? '' }}°</h3>
    </div>
    <div id="social">
      <!-- Open-Meteo también devuelve la hora de observación y zona horaria local. -->
      <h2>Zona horaria: {{ clima?.timezone || '' }}</h2>
      <h2>Hora de observación: {{ clima?.current?.time || '' }}</h2>
      <h1 id="emoji">😎</h1>
    </div>
  </section>
</template>
