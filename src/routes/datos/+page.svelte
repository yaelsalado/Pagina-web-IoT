<script lang="ts">
  import * as Table from "$lib/components/ui/table/index.js";
  import { onMount } from "svelte";

  let datosEstacion: any[] = [];
  let errorMsg = "";

  // URL de la API para obtener datos
  const API_URL = "http://localhost:5000/get_all";
  let controller: AbortController;

  // Función para obtener clima desde Open-Meteo
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

  // Función para cargar datos y actualizar tabla
  async function cargarDatos() {
    if (controller) controller.abort();
    controller = new AbortController();

    try {
      const res = await fetch(API_URL, { signal: controller.signal });
      if (!res.ok) throw new Error(res.statusText);

      const data = await res.json();
      if (!Array.isArray(data)) return;

      // Traemos el clima una sola vez para todas las filas
      const climaActual = await obtenerClimaOpenMeteo();

      // Mapear datos y ordenar de más nuevo a más viejo
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

  // Ejecutar al montar el componente
  onMount(() => {
    cargarDatos(); // cargar al inicio
    const interval = setInterval(cargarDatos, 2000); // actualizar cada 2s
    return () => clearInterval(interval);
  });
</script>

<div class="flex justify-center items-center min-h-[80vh] bg-gray-50 dark:bg-gray-900 px-4 py-6">
  <div class="w-full max-w-4xl rounded-3xl overflow-hidden shadow-xl border border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-800 transition-colors">
    
    <!-- Encabezado -->
    <div class="bg-gradient-to-r from-blue-500 to-indigo-500 dark:from-blue-700 dark:to-indigo-900 p-4">
      <h2 class="text-xl font-semibold text-white text-center">Registros de los sensores</h2>
    </div>

    <!-- Mensaje de error -->
    {#if errorMsg}
      <div class="text-red-500 text-center py-2">{errorMsg}</div>
    {/if}

    <!-- Tabla -->
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
  </div>
</div>
