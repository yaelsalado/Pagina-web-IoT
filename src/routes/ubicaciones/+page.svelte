<script lang="ts">
	import { onMount } from "svelte";
	import { setOptions, importLibrary } from "@googlemaps/js-api-loader";
	import { goto } from "$app/navigation";

	let map: any;

	const markers = [
		{
			id: "01",
			position: { lat: 20.735237741324806, lng: -103.45448702136937 },
			title: "Estación 1 - Tec GDL",
		},
		{
			id: "02",
			position: { lat: 20.65641926318562, lng: -103.39922049101062 },
			title: "Estación 2 - Chapalita",
		},
		{
			id: "03",
			position: { lat: 20.657560861676444, lng: -103.35111620635446 },
			title: "Estación 3 - Americana",
		},
	];

	onMount(async () => {
		setOptions({
			key: "AIzaSyCrxJJ0pLwyzEcxbnDkK5nuttRcWohUH9U",
			v: "weekly",
		});

		const { Map } = await importLibrary("maps");
		const { Marker } = await importLibrary("marker");

		const mapElement = document.getElementById("map") as HTMLElement;

		map = new Map(mapElement, {
			center: { lat: 20.70, lng: -103.40 },
			zoom: 12,
			mapId: "DEMO_MAP_ID",
			styles: [
				{ elementType: "geometry", stylers: [{ color: "#1e293b" }] },
				{ elementType: "labels.text.stroke", stylers: [{ color: "#1e293b" }] },
				{ elementType: "labels.text.fill", stylers: [{ color: "#f8fafc" }] },
				{
					featureType: "administrative.locality",
					elementType: "labels.text.fill",
					stylers: [{ color: "#f8fafc" }]
				},
				{
					featureType: "poi",
					elementType: "labels.text.fill",
					stylers: [{ color: "#f8fafc" }]
				},
				{
					featureType: "road",
					elementType: "geometry",
					stylers: [{ color: "#374151" }]
				},
				{
					featureType: "road",
					elementType: "labels.text.fill",
					stylers: [{ color: "#f8fafc" }]
				},
				{
					featureType: "water",
					elementType: "geometry",
					stylers: [{ color: "#0f172a" }]
				},
				{
					featureType: "water",
					elementType: "labels.text.fill",
					stylers: [{ color: "#f8fafc" }]
				}
			]
		});

		markers.forEach(({ id, position, title }) => {
			const marker = new Marker({
				position,
				map,
				title,
			});

			marker.addListener("click", () => {
				goto(`/estaciones?select=${id}`);
			});
		});
	});
</script>

<style>
	body {
		@apply bg-gradient-to-br from-gray-100 via-gray-50 to-gray-100 dark:from-gray-900 dark:via-gray-800 dark:to-gray-900 font-sans;
	}
</style>

<div class="min-h-screen flex flex-col items-center p-6">
	<!-- Encabezado -->
	<h1 class="text-3xl sm:text-4xl font-bold mb-2 text-center text-gray-900 dark:text-gray-100">
		Ubicación de nuestras estaciones 📍
	</h1>
	<p class="text-gray-600 dark:text-gray-300 mb-6 text-center max-w-xl">
		Haz click en cualquiera de los marcadores para ver información detallada de cada estación.
	</p>

	<!-- Mapa -->
	<div id="map" class="w-full max-w-4xl h-[60vh] rounded-xl shadow-lg border border-gray-200 dark:border-gray-700 overflow-hidden"></div>

	<!-- Tarjetas de estaciones -->
	<div class="grid grid-cols-1 sm:grid-cols-3 gap-4 mt-6 w-full max-w-5xl">
		{#each markers as { id, title }}
			<div
				class="p-4 bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100 rounded-lg shadow hover:shadow-lg cursor-pointer transition transform hover:-translate-y-1"
				on:click={() => goto(`/estaciones?select=${id}`)}
			>
				<h2 class="font-semibold text-lg">{title}</h2>
				<p class="text-gray-500 dark:text-gray-300 text-sm mt-1">Haz click para ver más detalles</p>
			</div>
		{/each}
	</div>
</div>
