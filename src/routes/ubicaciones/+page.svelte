<script lang="ts">
	import { onMount } from "svelte";
	import { setOptions, importLibrary } from "@googlemaps/js-api-loader";
	import { goto } from "$app/navigation";

	let map: any;

	onMount(async () => {
		// Configurar la API de Google Maps
		setOptions({
			key: "AIzaSyCrxJJ0pLwyzEcxbnDkK5nuttRcWohUH9U",
			v: "weekly",
		});

		const { Map } = await importLibrary("maps");
		const { Marker } = await importLibrary("marker");

		const mapElement = document.getElementById("map") as HTMLElement;

		map = new Map(mapElement, {
			center: { lat: 20.70, lng: -103.40 }, // centro aproximado entre estaciones
			zoom: 12,
			mapId: "DEMO_MAP_ID",
		});

		// Definir marcadores con sus ubicaciones y rutas
		const markers = [
			{
				position: { lat: 20.735237741324806, lng: -103.45448702136937 },
				title: "Estación 1 - Tec GDL",
				ruta: "/tec"
			},
			{
				position: { lat: 20.65641926318562, lng: -103.39922049101062 },
				title: "Estación 2 - Chapalita",
				ruta: "/chapalita"
			},
			{
				position: { lat: 20.657560861676444, lng: -103.35111620635446 },
				title: "Estación 3 - Americana",
				ruta: "/americana"
			},
		];

		// Crear los marcadores y añadir evento click
		markers.forEach(({ position, title, ruta }) => {
			const marker = new Marker({
				position,
				map,
				title,
			});

			// Redirigir según la estación
			marker.addListener("click", () => goto(ruta));
		});
	});
</script>

<style>
	#map {
		width: 70%;
		height: 50vh;
		border-radius: 1rem;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
		margin: 0 auto;
	}
</style>

<div class="p-6 flex flex-col items-center">
	<h1 class="text-2xl font-semibold mb-4">Ubicación de nuestras estaciones</h1>
	<div id="map"></div>
</div>
