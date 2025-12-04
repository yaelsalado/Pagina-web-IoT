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

            // Filtrar SOLO sensor 01 y mapear
            const nuevos = data
                .filter((row: any) => Number(row.sensor_id) === 1)
                .map((row: any) => {
                    const sensorHumedad = parseFloat(row.humedad as any) || 0;
                    let humedadPct = (1 - sensorHumedad / 4095) * 100;
                    humedadPct = Math.max(0, Math.min(100, humedadPct));

                    return {
                        _rowId: row.id,
                        id: row.sensor_id,
                        humedad: `${humedadPct.toFixed(1)} %`,
                        humedad_num: humedadPct,
                        altura_num: Number(row.altura),
                        altura: Number(row.altura).toFixed(2) + " cm",
                        created_at: row.created_at,
                        fecha: new Date(row.created_at).toLocaleDateString("es-MX", {
                            day: "2-digit",
                            month: "short",
                            year: "numeric"
                        })
                    };
                })
                .sort((a, b) => b._rowId - a._rowId);

            /* ---------------------------- */
            /*     TABLA: últimos 10       */
            /* ---------------------------- */
            datosEstacion = nuevos.slice(0, 10);

            /* ---------------------------------------------- */
            /*     GRÁFICO DE ALTURAS: últimos 10 registros   */
            /* ---------------------------------------------- */
            chartAltData = nuevos
                .slice(0, 10)
                .map((d) => ({
                    label: d.fecha,
                    altura: Math.round(d.altura_num * 100) / 100
                }))
                .reverse();

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
            prediccion = etiquetas[data.clase_predicha] ?? "Desconocido";

            let probs = data.probabilidades.slice(0, 4).map((v: any) => Number(v) || 0);
            const total = probs.reduce((s, x) => s + x, 0);
            if (total > 0) probs = probs.map((x) => x / total);
            probabilidades = probs;

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
        cargarDatos();
        cargarPrediccion();

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

        <div class="bg-gradient-to-r from-blue-500 to-indigo-500 dark:from-blue-700 dark:to-indigo-900 p-4">
            <h2 class="text-xl font-semibold text-white text-center">Registros de los Sensores — Sensor 01</h2>
        </div>

        <div class="overflow-x-auto">
            <Table.Root class="min-w-full">
                <Table.Header>
                    <Table.Row class="bg-gray-100 dark:bg-gray-700 text-gray-700 dark:text-gray-200 uppercase text-sm tracking-wider">
                        <Table.Head class="text-center py-2">ID Estación</Table.Head>
                        <Table.Head class="text-center py-2">Humedad</Table.Head>
                        <Table.Head class="text-center py-2">Altura</Table.Head>
                        <Table.Head class="text-center py-2">Fecha</Table.Head>
                    </Table.Row>
                </Table.Header>

                <Table.Body>
                    {#each datosEstacion as dato (dato._rowId)}
                        <Table.Row class="bg-gray-50 dark:bg-gray-800 hover:bg-blue-100 dark:hover:bg-blue-900/50">
                            <Table.Cell class="text-center py-2">{dato.id}</Table.Cell>
                            <Table.Cell class="text-center py-2">{dato.humedad}</Table.Cell>
                            <Table.Cell class="text-center py-2">{dato.altura}</Table.Cell>
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
                <h2 class="text-xl font-bold mb-2">🌤️ Predicción del clima para mañana</h2>

                {#if prediccion}
                    <p class="text-2xl font-semibold">{prediccion}</p>
                    <p class="text-gray-600 dark:text-gray-300 mt-1">
                        Probabilidad más alta: {(Math.max(...probabilidades) * 100).toFixed(1)}%
                    </p>
                {:else}
                    <p class="text-gray-500">Calculando predicción...</p>
                {/if}
            </div>
        </div>

        <!-- ========================== -->
        <!--     GRÁFICO PROBABILIDAD   -->
        <!-- ========================== -->
        <div class="px-6 pb-10">
            <div class="rounded-2xl shadow-md border border-gray-200 dark:border-gray-700 bg-gray-50 dark:bg-gray-900 p-6">
                <h2 class="text-xl font-bold mb-4">📊 Distribución visual del pronóstico</h2>

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
