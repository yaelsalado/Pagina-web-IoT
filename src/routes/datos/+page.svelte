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

            // Asegurar que sea un array
            if (!Array.isArray(data)) return;

            // Mapear todos los registros
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

<!-- UI REAL -->
<div class="flex justify-center items-center min-h-[80vh] bg-background px-4">
    <div
        class="w-full max-w-3xl rounded-2xl overflow-hidden shadow-lg border border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-900 transition-colors"
    >
        <Table.Root>
            <Table.Caption class="text-center font-semibold py-3 text-gray-800 dark:text-gray-200">
                Historial completo de registros
            </Table.Caption>

            <Table.Header>
                <Table.Row class="bg-gray-100 dark:bg-gray-800">
                    <Table.Head class="text-center">ID Estación</Table.Head>
                    <Table.Head class="text-center">Humedad</Table.Head>
                    <Table.Head class="text-center">Altura</Table.Head>
                    <Table.Head class="text-center">Clima</Table.Head>
                    <Table.Head class="text-center">Fecha</Table.Head>
                </Table.Row>
            </Table.Header>

            <Table.Body>
                {#each datosEstacion as dato (dato._rowId)}
                    <Table.Row class="hover:bg-gray-50 dark:hover:bg-gray-800/50 transition-colors">
                        <Table.Cell class="text-center">{dato.id}</Table.Cell>
                        <Table.Cell class="text-center">{dato.humedad}</Table.Cell>
                        <Table.Cell class="text-center">{dato.altura}</Table.Cell>
                        <Table.Cell class="text-center">{dato.clima}</Table.Cell>
                        <Table.Cell class="text-center">{dato.fecha}</Table.Cell>
                    </Table.Row>
                {/each}
            </Table.Body>
        </Table.Root>
    </div>
</div>
