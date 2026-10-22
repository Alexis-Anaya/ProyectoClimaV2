<script setup>
    const parametros = new URLSearchParams({
                q: ciudad,
                format: 'jsonv2'
            });

            try {
                const lugares = await solicitarJson(
                    `https://nominatim.openstreetmap.org/search?${parametros}`,
                    'Nominatim'
                );

                if (!Array.isArray(lugares) || lugares.length === 0) {
                    errorElemento.innerText = 'Ciudad no encontrada.';
                    return;
                }
</script>