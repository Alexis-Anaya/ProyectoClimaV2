<script>
    document.getElementById('ciudad').innerText = `Clima en ${lugar.display_name}`;
                document.getElementById('temperatura').innerText = `Temperatura Actual: ${clima.current.temperature_2m}°C`;
                document.getElementById('STermica').innerText = `Sensación térmica: ${clima.current.apparent_temperature}°C`;
                document.getElementById('humedad').innerText = `Humedad: ${clima.current.relative_humidity_2m}%`;
                document.getElementById('CMeteorologica').innerText = `Condición Meteorológica: ${clima.current.weather_code}`;
                document.getElementById('VViento').innerText = `Velocidad del viento: ${clima.current.wind_speed_10m} km/h`;
                document.getElementById('latitud').innerText = `Latitud: ${clima.latitude}°`;
                document.getElementById('longitud').innerText = `Longitud: ${clima.longitude}°`;
</script>