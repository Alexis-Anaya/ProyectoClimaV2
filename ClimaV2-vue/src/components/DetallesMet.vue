<script setup>
    const lugar = lugares[0];
                const apiUrl = new URL('https://api.open-meteo.com/v1/forecast');
                apiUrl.search = new URLSearchParams({
                    latitude: lugar.lat,
                    longitude: lugar.lon,
                    current: 'relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m,temperature_2m'
                });

                const clima = await solicitarJson(apiUrl, 'Open-Meteo');
                if (!clima.current) {
                    throw new ApiError(
                        'El servicio meteorológico devolvió datos incompletos.',
                        'invalid-response'
                    );
                }
</script>