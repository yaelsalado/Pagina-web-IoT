<script lang="ts">
    import * as Table from "$lib/components/ui/table/index.js";
    import { onMount } from "svelte";

    let datosEstacion: any[] = [];
    const API_URL = "http://localhost:5000/get_all";

    onMount(() => {
        cargarDatos();
        const interval = setInterval(cargarDatos, 2000);
        return () => clearInterval(interval);
    });

    async function cargarDatos() {
        try {
            const res = await fetch(API_URL);
            const data = await res.json();
            if (!Array.isArray(data)) return;

            datosEstacion = data.map(row => ({
                _rowId: row.id,
                id: row.sensor_id,
                humedad: row.humedad + "%",
                altura: row.altura.toFixed(2) + " m",
                clima: row.clima,
                fecha: new Date(row.created_at).toLocaleDateString("es-MX", {
                    day: "2-digit",
                    month: "short",
                    year: "numeric"
                })
            }));

        } catch (error) {
            console.error("❌ Error al actualizar datos:", error);
        }
    }
</script>

<div class="flex justify-center items-center min-h-[80vh] bg-gray-50 dark:bg-gray-900 px-4 py-6">
    <div
        class="w-full max-w-4xl rounded-3xl overflow-hidden shadow-xl border border-gray-200 dark:border-gray-700
               bg-white dark:bg-gray-800 transition-colors"
    >
        <!-- Encabezado -->
        <div class="bg-gradient-to-r from-blue-500 to-indigo-500 dark:from-blue-700 dark:to-indigo-900 p-4">
            <h2 class="text-xl font-semibold text-white text-center">Registros de los sensores</h2>
        </div>

        <!-- ⭐ Contenedor responsivo para la tabla -->
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
                        <Table.Row
                            class="even:bg-gray-50 odd:bg-gray-100 dark:even:bg-gray-800 dark:odd:bg-gray-700
                                   hover:bg-blue-100 dark:hover:bg-blue-900/50 transition-colors"
                        >
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
    </div>
</div>
