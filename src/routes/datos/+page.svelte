<script lang="ts">
	import * as Table from "$lib/components/ui/table/index.js";

	function generarDatos() {
		const climas = ["Soleado", "Nublado", "Lluvioso", "Ventoso", "Tormenta"];
		const datos = [];

		for (let i = 0; i < 7; i++) {
			const fecha = new Date();
			fecha.setDate(fecha.getDate() - i);

			datos.push({
				id: `EST-${100 + i}`,
				humedad: `${Math.floor(Math.random() * 40) + 40}%`,
				altura: (13.8 + Math.random() * 1.2).toFixed(2) + " m",
				clima: climas[Math.floor(Math.random() * climas.length)],
				fecha: fecha.toLocaleDateString("es-MX", {
					day: "2-digit",
					month: "short",
					year: "numeric"
				})
			});
		}
		return datos;
	}

	const datosEstacion = generarDatos();
</script>

<div class="flex justify-center items-center min-h-[80vh] bg-background px-4">
	<div
		class="w-full max-w-3xl rounded-2xl overflow-hidden shadow-lg border border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-900 transition-colors"
	>
		<Table.Root>
			<Table.Caption class="text-center font-semibold py-3 text-gray-800 dark:text-gray-200">
				Datos recientes de la estación de monitoreo
			</Table.Caption>

			<Table.Header>
				<Table.Row class="bg-gray-100 dark:bg-gray-800">
					<Table.Head class="w-[120px] text-center text-gray-900 dark:text-gray-100">
						ID Estación
					</Table.Head>
					<Table.Head class="text-center text-gray-900 dark:text-gray-100">Humedad</Table.Head>
					<Table.Head class="text-center text-gray-900 dark:text-gray-100">Altura</Table.Head>
					<Table.Head class="text-center text-gray-900 dark:text-gray-100">Clima</Table.Head>
					<Table.Head class="text-center text-gray-900 dark:text-gray-100">Fecha</Table.Head>
				</Table.Row>
			</Table.Header>

			<Table.Body>
				{#each datosEstacion as dato (dato.id)}
					<Table.Row
						class="hover:bg-gray-50 dark:hover:bg-gray-800/50 transition-colors"
					>
						<Table.Cell class="font-medium text-center text-gray-800 dark:text-gray-200">
							{dato.id}
						</Table.Cell>
						<Table.Cell class="text-center text-gray-700 dark:text-gray-300">
							{dato.humedad}
						</Table.Cell>
						<Table.Cell class="text-center text-gray-700 dark:text-gray-300">
							{dato.altura}
						</Table.Cell>
						<Table.Cell class="text-center text-gray-700 dark:text-gray-300">
							{dato.clima}
						</Table.Cell>
						<Table.Cell class="text-center text-gray-700 dark:text-gray-300">
							{dato.fecha}
						</Table.Cell>
					</Table.Row>
				{/each}
			</Table.Body>
		</Table.Root>
	</div>
</div>
