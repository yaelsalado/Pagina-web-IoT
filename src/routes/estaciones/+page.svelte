<script lang="ts">
	import { onMount } from "svelte";
	import * as Select from "$lib/components/ui/select/index.js";

	// 🔔 Nuevos imports para la alerta pop-up
	import * as Alert from "$lib/components/ui/alert/index.js";
	import AlertCircleIcon from "@lucide/svelte/icons/alert-circle";

	let Altura = "Esperando datos...";
	let Humedad = "Esperando datos...";
	let alerta = "";
	let colorAlerta = "";

	let selectedStation = "01";

	// 🔔 Estado del popup
	let mostrarPopup = false;

	// Controlar popup cuando colorAlerta es rojo
	$: if (colorAlerta === "rojo") {
	    mostrarPopup = true;

	    // 📱 Vibración para celulares compatibles
	    if (navigator.vibrate) {
	        navigator.vibrate([200, 120, 200]);
	    }

	    setTimeout(() => {
	        mostrarPopup = false;
	    }, 3500);
	}

	onMount(() => {
		const params = new URLSearchParams(window.location.search);
		const id = params.get("select");
		if (id) selectedStation = id;

		obtenerDatos();
		const intervalo = setInterval(obtenerDatos, 1000);
		return () => clearInterval(intervalo);
	});

	function buildURL() {
		return `http://10.179.97.220:5000/get_sensor_${selectedStation}`;
	}

	function verificarAltura(altura: number) {
		if (altura >= 1.45) {
			alerta = "🚫 No pase: nivel de agua crítico";
			colorAlerta = "rojo"; // → Activará popup
		} else if (altura >= 1.35) {
			alerta = "⚠ Precaución: el nivel de agua está subiendo";
			colorAlerta = "amarillo";
		} else {
			alerta = "✅ Nivel de agua seguro";
			colorAlerta = "verde";
		}
	}

	function interpretarHumedad(humedad: number) {
		if (humedad >= 80) return `Lluvia detectada (${humedad.toFixed(1)}%)`;
		if (humedad >= 60) return `Humedad alta (${humedad.toFixed(1)}%)`;
		return `Ambiente seco (${humedad.toFixed(1)}%)`;
	}

	async function obtenerDatos() {
		try {
			const res = await fetch(buildURL());
			if (!res.ok) throw new Error("Respuesta no válida del servidor");

			const arr = await res.json();
			const data = arr[0];
			if (!data) return;

			// 🔹 Altura: sensor envía cm → convertir a metros
			const alturaCm = parseFloat(data.altura);
			const alturaMetros = alturaCm / 100;

			// 🔹 Humedad inversa: 0 → 100%, 4095 → 0%
			const sensorHumedad = parseFloat(data.humedad);
			const humedad = (1 - sensorHumedad / 4095) * 100;

			verificarAltura(alturaMetros);
			Altura = `${alturaMetros.toFixed(2)} m`;
			Humedad = interpretarHumedad(humedad);
		} catch (err) {
			console.error(err);
			alerta = "Error al conectar con el servidor  =(";
			colorAlerta = "azul"; // popup NO aparece por errores
		}
	}
</script>

<div class="min-h-[80vh] flex flex-col items-center justify-start gap-6 py-6 px-4 bg-gray-50 dark:bg-gray-900 transition-colors">

	<h1 class="text-2xl md:text-3xl font-bold text-gray-800 dark:text-gray-100 mb-2">
		Lectura de Sensores
	</h1>

	<!-- Selector -->
	<div class="w-full max-w-xs">
		<Select.Root type="single" bind:value={selectedStation} on:change={obtenerDatos}>
			<Select.Trigger class="w-full font-semibold text-gray-800 dark:text-gray-200 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-700 px-4 py-3 rounded-lg shadow-sm hover:shadow-md transition-shadow">
				Seleccionar estación
			</Select.Trigger>

			<Select.Content class="bg-white dark:bg-gray-800 text-gray-800 dark:text-gray-200 border border-gray-300 dark:border-gray-700 rounded-lg shadow-md w-full">
				<Select.Item value="01" class="px-3 py-2 hover:bg-gray-200 dark:hover:bg-gray-700 rounded-md">TEC</Select.Item>
				<Select.Item value="02" class="px-3 py-2 hover:bg-gray-200 dark:hover:bg-gray-700 rounded-md">CHAPALITA</Select.Item>
				<Select.Item value="03" class="px-3 py-2 hover:bg-gray-200 dark:hover:bg-gray-700 rounded-md">AMERICANA</Select.Item>
			</Select.Content>
		</Select.Root>
	</div>

	<!-- 🔔 POP-UP ABAJO CENTRO (solo nivel crítico) -->
	{#if mostrarPopup}
		<div
			class="fixed bottom-6 left-1/2 -translate-x-1/2 z-50 max-w-lg w-[92%] md:w-[380px]"
		>
			<Alert.Root variant="destructive" class="w-full shadow-xl rounded-xl">
				<AlertCircleIcon />
				<Alert.Title>Tome rutas alternas</Alert.Title>
				<Alert.Description>
					El nivel de agua es crítico y podría haber riesgo de inundación.
				</Alert.Description>
			</Alert.Root>
		</div>
	{/if}

	<!-- Alerta grande -->
<div class="max-w-md w-full py-3 px-6 rounded-xl font-semibold text-center shadow-md
    {colorAlerta === 'verde' ? 'bg-green-100 text-green-800 dark:bg-green-900/50 dark:text-green-400 border border-green-400' : ''}
    {colorAlerta === 'amarillo' ? 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/50 dark:text-yellow-400 border border-yellow-400' : ''}
    {colorAlerta === 'rojo' ? 'bg-red-100 text-red-800 dark:bg-red-900/50 dark:text-red-400 border border-red-400' : ''}
    {colorAlerta === 'azul' ? 'bg-blue-100 text-blue-800 dark:bg-blue-900/50 dark:text-blue-400 border border-blue-400' : ''}">
    <p class="text-lg">{alerta}</p>
</div>


	<!-- Cards -->
	<div class="grid grid-cols-1 sm:grid-cols-2 gap-6 w-full max-w-xl">

		<div class="bg-white dark:bg-gray-800 text-gray-800 dark:text-gray-100 p-6 rounded-xl shadow-md flex flex-col items-center">
			<h2 class="text-lg font-medium mb-2">Altura del agua</h2>
			<p class="text-3xl font-bold">{Altura}</p>
		</div>

		<div class="bg-white dark:bg-gray-800 text-gray-800 dark:text-gray-100 p-6 rounded-xl shadow-md flex flex-col items-center">
			<h2 class="text-lg font-medium mb-2">Condiciones</h2>
			<p class="text-2xl font-bold break-words text-center">{Humedad}</p>
		</div>

	</div>
</div>

<p class="mt-4 text-sm text-gray-500 dark:text-gray-400 text-center">
	Estación seleccionada: {selectedStation === "01" ? "TEC" : selectedStation === "02" ? "CHAPALITA" : "AMERICANA"}
</p>

<style>
	h1, h2, p {
		transition: color 0.2s ease;
	}
</style>
