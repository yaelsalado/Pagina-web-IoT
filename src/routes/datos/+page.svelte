<script lang="ts">
    import * as Table from "$lib/components/ui/table/index.js";
    import * as Chart from "$lib/components/ui/chart/index.js";
    import { scaleBand } from "d3-scale";
    import { BarChart } from "layerchart";
    import { onMount } from "svelte";

    /* ------------------------------ */
    /*   VARIABLES PRINCIPALES        */
    /* ------------------------------ */
    let datosEstacion: any[] = [];
    let errorMsg = "";
    let controller: AbortController | null = null;

    const API_URL = "http://localhost:5000/get_all";

    // Predicción para mañana
    let prediccion = "";
    let probabilidades: number[] = [0, 0, 0, 0];

    /* ------------------------------ */
    /*       CONFIG GRÁFICO (prob)    */
    /* ------------------------------ */
    let chartData = [
        { label: "Soleado", prob: 0 },
        { label: "Nublado", prob: 0 },
        { label: "Lluvioso", prob: 0 },
        { label: "Tormenta", prob: 0 }
    ];

    const chartConfig = {
        prob: {
            label: "Probabilidad (%)",
            color: "#2563eb"
        }
    } satisfies Chart.ChartConfig;

    function actualizarChart() {
        chartData = [
            { label: "Soleado", prob: (probabilidades[0] || 0) * 100 },
            { label: "Nublado", prob: (probabilidades[1] || 0) * 100 },
            { label: "Lluvioso", prob: (probabilidades[2] || 0) * 100 },
            { label: "Tormenta", prob: (probabilidades[3] || 0) * 100 }
        ];
    }

    /* ------------------------------ */
    /*   CONFIG GRÁFICO ALTURAS       */
    /* ------------------------------ */
    let chartAltData: { label: string; altura: number }[] = [];

    const chartAltConfig = {
        altura: {
            label: "Altura (cm)",
            color: "#10b981"
        }
    } satisfies Chart.ChartConfig;

    /* ------------------------------ */
    /*   CLIMA OPEN-METEO (opcional)  */
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

            // Obtener clima una sola vez (para mostrar en la tabla)
            const climaActual = await obtenerClimaOpenMeteo();

            // Filtrar SOLO sensor 01 y mapear datos
            const nuevos = data
                .filter((row: any) => Number(row.sensor_id) === 1) // <<--- SOLO SENSOR 01
                .map((row: any) => ({
                    _rowId: row.id,
                    id: row.sensor_id,
                    humedad: row.humedad + "",
                    altura_num: Number(row.altura), 
                    altura: (Number(row.altura)).toFixed(2) + " cm",
                    clima: climaActual,
                    created_at: row.created_at,
                    fecha: new Date(row.created_at).toLocaleDateString("es-MX", {
                        day: "2-digit",
                        month: "short",
                        year: "numeric"
                    })
                }))
                .sort((a, b) => b._rowId - a._rowId); // De más reciente → más viejo

            /* ---------------------------- */
            /*     TABLA: últimos 20       */
            /* ---------------------------- */
            datosEstacion = nuevos.slice(0, 20);
            errorMsg = "";

            /* ---------------------------------------------- */
            /*     GRÁFICO DE ALTURAS: últimos 10 registros   */
            /* ---------------------------------------------- */
            chartAltData = nuevos
                .slice(0, 10) // <<--- SOLO 10
                .map((d) => ({
                    label: d.fecha,
                    altura: Math.round(d.altura_num * 100) / 100
                }))
                .reverse(); // Orden cronológico (antiguo → reciente)

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
            if (!res.ok) throw new Error(await res.text());
            const data = await res.json();

            const etiquetas = ["Soleado", "Nublado", "Lluvioso", "Tormenta"];
            prediccion = etiquetas[data.clase_predicha] ?? "Desconocido";

            // normalizar probabilidades si vienen mal formadas
            if (Array.isArray(data.probabilidades) && data.probabilidades.length >= 4) {
                let probs = data.probabilidades.slice(0, 4).map((v: any) => Number(v) || 0);
                const total = probs.reduce((s: number, x: number) => s + x, 0);
                if (total > 0) probs = probs.map((x: number) => x / total);
                probabilidades = probs;
            } else {
                probabilidades = [0.25, 0.25, 0.25, 0.25];
            }

            actualizarChart();
        } catch (err) {
            console.error("❌ Error predicción clima:", err);
            probabilidades = [0.25, 0.25, 0.25, 0.25];
            actualizarChart();
        }
    }

    /* ------------------------------ */
    /*   ON MOUNT                     */
    /* ------------------------------ */
    onMount(() => {
        // cargar inicialmente
        cargarDatos();
        cargarPrediccion();

        // refrescar c/2s (como tenías antes)
        const interval = setInterval(() => {
            cargarDatos();
            cargarPrediccion();
        }, 2000);

        return () => {
            clearInterval(interval);
            if (controller) controller.abort();
        };
    });
</script>

<!-- ========================== -->
<!--         TABLA DATOS        -->
<!-- ========================== -->
<div class="flex justify-center items-center min-h-[80vh] bg-gray-50 dark:bg-gray-900 px-4 py-6">
    <div class="w-full max-w-4xl rounded-3xl overflow-hidden shadow-xl border border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-800 transition-colors">

        <!-- Encabezado -->
        <div class="bg-gradient-to-r from-blue-500 to-indigo-500 dark:from-blue-700 dark:to-indigo-900 p-4">
            <h2 class="text-xl font-semibold text-white text-center">Registros de los Sensores — Sensor 01</h2>
        </div>

        {#if errorMsg}
            <div class="text-red-500 text-center py-2">{errorMsg}</div>
        {/if}

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
        <!--   GRÁFICO ALTURAS (últimos 10) -->
        <!-- ========================== -->
        <div class="px-6 pt-6">
            <div class="rounded-2xl shadow-md border border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-900 p-6">
                <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-4">
                    📏 Altura — Últimos 10 registros (Sensor 01)
                </h2>

                {#if chartAltData && chartAltData.length > 0}
                    <div class="w-full h-56">
                        <Chart.Container config={chartAltConfig} class="w-full h-full">
                            <BarChart
                                data={chartAltData}
                                xScale={scaleBand().padding(0.2)}
                                x="label"
                                axis="x"
                                seriesLayout="group"
                                legend={false}
                                series={[
                                    {
                                        key: "altura",
                                        label: chartAltConfig.altura.label,
                                        color: chartAltConfig.altura.color
                                    }
                                ]}
                            >
                                {#snippet tooltip()}
                                    <Chart.Tooltip />
                                {/snippet}
                            </BarChart>
                        </Chart.Container>
                    </div>
                {:else}
                    <p class="text-gray-500">No hay datos de altura para mostrar.</p>
                {/if}
            </div>
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
                                Probabilidad más alta:
                                {(Math.max(...probabilidades) * 100).toFixed(1)}%
                            </p>
                        </div>
                    </div>

                    <h3 class="mt-6 font-medium">Distribución:</h3>
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

        <!-- ========================== -->
        <!--     GRÁFICO PROBABILIDAD   -->
        <!-- ========================== -->
        <div class="px-6 pb-10">
            <div class="rounded-2xl shadow-md border border-gray-200 dark:border-gray-700 bg-gray-50 dark:bg-gray-900 p-6">
                <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-4">
                    📊 Distribución visual del pronóstico
                </h2>

                <div class="w-full h-60 bg-white dark:bg-gray-800 p-4 rounded-lg shadow-sm border border-gray-100 dark:border-gray-700">
                    <Chart.Container config={chartConfig} class="w-full h-full">
                        <BarChart
                            data={chartData}
                            xScale={scaleBand().padding(0.25)}
                            x="label"
                            axis="x"
                            seriesLayout="group"
                            legend
                            series={[
                                {
                                    key: "prob",
                                    label: chartConfig.prob.label,
                                    color: chartConfig.prob.color
                                }
                            ]}
                        >
                            {#snippet tooltip()}
                                <Chart.Tooltip />
                            {/snippet}
                        </BarChart>
                    </Chart.Container>
                </div>
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
