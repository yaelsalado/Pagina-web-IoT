<script lang="ts">
    import * as Table from "$lib/components/ui/table/index.js";
    import { onMount } from "svelte";

    /* ------------------------------ */
    /*   VARIABLES PRINCIPALES        */
    /* ------------------------------ */
    let datosEstacion: any[] = [];
    let errorMsg = "";
    let controller: AbortController;

    const API_URL = "http://localhost:5000/get_all";

    // Predicción para mañana
    let prediccion = "";
    let probabilidades: number[] = [];

    // Pronóstico 24 horas
    let forecast24 = [];

    /* ------------------------------ */
    /*   CLIMA OPEN-METEO             */
    /* ------------------------------ */
    async function obtenerClimaOpenMeteo(): Promise<string> {
        try {
            const lat = 20.735237741324806;
            const lon = -103.45448702136937;

            const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true`;
            const res = await fetch(url);
            if (!res.ok) return "Error clima";

            const data = await res.json();
            const code = data.current_weather?.weathercode;

            const mapCode: Record<number, string> = {
                0: "Despejado",
                1: "Parcialmente nublado",
                2: "Nublado",
                3: "Lluvia ligera",
                4: "Lluvia moderada",
                5: "Lluvia intensa",
                6: "Tormenta",
                7: "Nieve",
                8: "Niebla"
            };

            return mapCode[code] || "Desconocido";
        } catch (err) {
            console.error(err);
            return "Error clima";
        }
    }

    /* ------------------------------ */
    /*   CARGAR DATOS DE SENSORES     */
    /* ------------------------------ */
    async function cargarDatos() {
        if (controller) controller.abort();
        controller = new AbortController();

        try {
            const res = await fetch(API_URL, { signal: controller.signal });
            if (!res.ok) throw new Error(res.statusText);

            const data = await res.json();
            if (!Array.isArray(data)) return;

            // Obtener clima una sola vez
            const climaActual = await obtenerClimaOpenMeteo();

            // Mapear datos y ordenar
            const nuevos = data
                .map((row: any) => ({
                    _rowId: row.id,
                    id: row.sensor_id,
                    humedad: row.humedad + "",
                    altura: row.altura.toFixed(2) + " cm",
                    clima: climaActual,
                    fecha: new Date(row.created_at).toLocaleDateString("es-MX", {
                        day: "2-digit",
                        month: "short",
                        year: "numeric"
                    })
                }))
                .sort((a, b) => b._rowId - a._rowId);

            datosEstacion = nuevos.slice(0, 20);
            errorMsg = "";
        } catch (error: any) {
            if (error.name !== "AbortError") {
                console.error("❌ Error al actualizar datos:", error);
                errorMsg = "No se pudieron cargar los datos";
            }
        }
    }

    /* ------------------------------ */
    /*   PREDICCIÓN PARA MAÑANA       */
    /* ------------------------------ */
    async function cargarPrediccion() {
        try {
            const res = await fetch("http://localhost:5000/predict_tomorrow");
            const data = await res.json();

            const etiquetas = ["Soleado", "Nublado", "Lluvioso", "Tormenta"];
            prediccion = etiquetas[data.clase_predicha];
            probabilidades = data.probabilidades;
        } catch (err) {
            console.error("❌ Error predicción clima:", err);
        }
    }


    /* ------------------------------ */
    /*   ON MOUNT                     */
    /* ------------------------------ */
    onMount(() => {
        cargarDatos();
        cargarPrediccion();

        const interval = setInterval(() => {
            cargarDatos();
            cargarPrediccion();
        }, 2000);

        return () => clearInterval(interval);
    });
</script>

<!-- ========================== -->
<!--         TABLA DATOS        -->
<!-- ========================== -->
<div class="flex justify-center items-center min-h-[80vh] bg-gray-50 dark:bg-gray-900 px-4 py-6">
    <div class="w-full max-w-4xl rounded-3xl overflow-hidden shadow-xl border border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-800 transition-colors">

        <!-- Encabezado -->
        <div class="bg-gradient-to-r from-blue-500 to-indigo-500 dark:from-blue-700 dark:to-indigo-900 p-4">
            <h2 class="text-xl font-semibold text-white text-center">Registros de los Sensores</h2>
        </div>

        {#if errorMsg}
            <div class="text-red-500 text-center py-2">{errorMsg}</div>
        {/if}

        <!-- TABLA -->
        <div class="overflow-x-auto">
            <Table.Root class="min-w-full">
                <Table.Header>
                    <Table.Row class="bg-gray-100 dark:bg-gray-700 text-gray-700 dark:text-gray-200 uppercase text-sm tracking-wider">
                        <Table.Head class="text-center py-2">ID Estación</Table.Head>
                        <Table.Head class="text-center py-2">Humedad</Table.Head>
                        <Table.Head class="text-center py-2">Altura</Table.Head>
                        <Table.Head class="text-center py-2">Clima</Table.Head>
                        <Table.Head class="text-center py-2">Fecha</Table.Head>
                    </Table.Row>
                </Table.Header>

                <Table.Body>
                    {#each datosEstacion as dato (dato._rowId)}
                        <Table.Row class="bg-gray-50 dark:bg-gray-800 hover:bg-blue-100 dark:hover:bg-blue-900/50">
                            <Table.Cell class="text-center py-2">{dato.id}</Table.Cell>
                            <Table.Cell class="text-center py-2">{dato.humedad}</Table.Cell>
                            <Table.Cell class="text-center py-2">{dato.altura}</Table.Cell>
                            <Table.Cell class="text-center py-2">{dato.clima}</Table.Cell>
                            <Table.Cell class="text-center py-2">{dato.fecha}</Table.Cell>
                        </Table.Row>
                    {/each}
                </Table.Body>
            </Table.Root>
        </div>

        <!-- ========================== -->
        <!--   PREDICCIÓN MAÑANA       -->
        <!-- ========================== -->
        <div class="p-6">
            <div class="rounded-2xl shadow-md border border-gray-200 dark:border-gray-700 bg-gray-50 dark:bg-gray-900 p-6">
                <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 flex items-center gap-2">
                    🌤️ Predicción del clima para mañana
                </h2>

                {#if prediccion}
                    <div class="mt-4 flex items-center gap-4">
                        <div class="text-5xl">
                            {#if prediccion === "Soleado"} ☀️
                            {:else if prediccion === "Nublado"} ☁️
                            {:else if prediccion === "Lluvioso"} 🌧️
                            {:else} ⛈️ {/if}
                        </div>

                        <div>
                            <p class="text-2xl font-semibold">{prediccion}</p>
                            <p class="text-gray-600 dark:text-gray-300">
                                Probabilidad más alta: {(Math.max(...probabilidades) * 100).toFixed(1)}%
                            </p>
                        </div>
                    </div>

                    <h3 class="mt-4 font-medium">Distribución:</h3>
                    <ul class="mt-2 space-y-1">
                        <li>☀️ Soleado: {(probabilidades[0] * 100).toFixed(1)}%</li>
                        <li>☁️ Nublado: {(probabilidades[1] * 100).toFixed(1)}%</li>
                        <li>🌧️ Lluvioso: {(probabilidades[2] * 100).toFixed(1)}%</li>
                        <li>⛈️ Tormenta: {(probabilidades[3] * 100).toFixed(1)}%</li>
                    </ul>
                {:else}
                    <p class="mt-3 text-gray-500">Calculando predicción...</p>
                {/if}
            </div>
        </div>
        
    </div>
</div>

<style>
    .no-scrollbar::-webkit-scrollbar {
        display: none;
    }
    .no-scrollbar {
        -ms-overflow-style: none;
        scrollbar-width: none;
    }
</style>
