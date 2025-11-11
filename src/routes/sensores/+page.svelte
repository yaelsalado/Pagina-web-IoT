<script lang="ts">
	import { onMount } from "svelte";

	let Altura = "Esperando datos...";
	let Humedad = "Esperando datos...";
	let alerta = "";
	let colorAlerta = "";

	const API_URL = "http://localhost:5000/get_data/clase_Esp32";

	function verificarAltura(altura: number) {
		if (altura >= 0.145) {
			alerta = "✅ Nivel de agua seguro";
			colorAlerta = "verde";
		} else if (altura >= 0.135 && altura < 0.145) {
			alerta = "⚠ Precaución: el nivel de agua está subiendo";
			colorAlerta = "amarillo";
		} else {
			alerta = "🚫 No pase: nivel de agua crítico";
			colorAlerta = "rojo";
		}
	}

	function interpretarHumedad(humedad: number) {
		if (humedad >= 80) {
			return `Lluvia detectada (${humedad.toFixed(1)}%)`;
		} else if (humedad >= 60) {
			return `Humedad alta (${humedad.toFixed(1)}%)`;
		} else {
			return `Ambiente seco (${humedad.toFixed(1)}%)`;
		}
	}

	async function obtenerDatos() {
		try {
			const res = await fetch(API_URL);
			if (!res.ok) throw new Error("Respuesta no válida del servidor");
			const sensorData = await res.json();

			const distancia = parseFloat(sensorData.Distancia);
			const lluvia = parseFloat(sensorData.Lluvia);
			const alturaMetros = distancia / 100;
			const humedad = 100 - (lluvia / 4095) * 100;

			verificarAltura(alturaMetros);
			Altura = `Altura: ${alturaMetros.toFixed(2)} m`;
			Humedad = interpretarHumedad(humedad);
		} catch (err) {
			console.error("Error al obtener datos:", err);
			alerta = "❌ Error al conectar con el servidor";
			colorAlerta = "rojo";
		}
	}

	onMount(() => {
		obtenerDatos();
		const intervalo = setInterval(obtenerDatos, 1500);
		return () => clearInterval(intervalo);
	});
</script>

<div class="page">
	<h1 class="titulo">Lectura de Sensores</h1>

	<div class="alerta {colorAlerta}">
		<p>{alerta}</p>
	</div>

	<div class="cards">
		<div class="card">
			<h2>Altura del agua</h2>
			<p>{Altura}</p>
		</div>

		<div class="card">
			<h2>Condiciones</h2>
			<p>{Humedad}</p>
		</div>
	</div>
</div>

<style>
	.page {
		min-height: 80vh;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		background: var(--background);
		color: var(--foreground);
		font-family: 'Inter', sans-serif;
		padding: 2rem;
		text-align: center;
		transition: background 0.3s ease, color 0.3s ease;
	}

	.titulo {
		font-size: 1.8rem;
		font-weight: 700;
		margin-bottom: 1.5rem;
	}

	.alerta {
		padding: 1rem 2rem;
		border-radius: 0.75rem;
		font-weight: 600;
		text-align: center;
		margin-bottom: 2rem;
		max-width: 400px;
		width: 100%;
	}

	.alerta.verde {
		background-color: rgba(34, 197, 94, 0.15);
		color: rgb(34, 197, 94);
		border: 1px solid rgb(34, 197, 94);
	}

	.alerta.amarillo {
		background-color: rgba(250, 204, 21, 0.15);
		color: rgb(250, 204, 21);
		border: 1px solid rgb(250, 204, 21);
	}

	.alerta.rojo {
		background-color: rgba(239, 68, 68, 0.15);
		color: rgb(239, 68, 68);
		border: 1px solid rgb(239, 68, 68);
	}

	.cards {
		display: flex;
		gap: 1.5rem;
		flex-wrap: wrap;
		justify-content: center;
	}

	.card {
		background-color: var(--card);
		color: var(--card-foreground);
		padding: 1.5rem 2rem;
		border-radius: 1rem;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
		width: 240px;
		text-align: center;
		transition: transform 0.2s ease, box-shadow 0.2s ease;
	}

	.card:hover {
		transform: translateY(-4px);
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
	}

	.card h2 {
		font-size: 1.1rem;
		margin-bottom: 0.5rem;
		color: var(--muted-foreground);
	}

	.card p {
		font-size: 1rem;
		font-weight: 500;
	}
</style>
