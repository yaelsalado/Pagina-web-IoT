<script>
	import portada from "$lib/assets/img2.png";
	import secundaria from "$lib/assets/img3.png";
</script>

<!-- 🔹 Sección principal (imagen izquierda, texto derecha) -->
<div class="flex flex-col md:flex-row items-center justify-center gap-10 mt-10 px-6 md:px-12 transition-colors duration-300">

	<!-- Imagen -->
	<img
		src={portada}
		alt="Imagen principal"
		class="
			rounded-2xl shadow-lg
			w-full max-w-xs           
			md:max-w-md md:w-[40%]   
			object-cover
		"
	/>

	<!-- Texto -->
	<div class="text-center md:text-left md:w-[45%] px-2 md:px-0">
		<h2 class="text-2xl md:text-3xl font-semibold mb-4 text-gray-800 dark:text-gray-100 transition-colors duration-300">
			Bienvenido a TeCuidamos
		</h2>

		<p class="text-gray-700 dark:text-gray-300 text-base md:text-lg leading-relaxed transition-colors duration-300">
			Nuestro sistema monitorea en tiempo real los niveles de agua y las condiciones ambientales 
			para mantenerte informado y prevenir riesgos. Gracias a nuestras estaciones distribuidas, 
			podemos ofrecerte datos precisos y actualizados desde cualquier ubicación.
		</p>
	</div>
</div>

<div class="mt-10 px-6 md:px-12">

  <!-- Título de sección -->
  <h2 class="text-2xl md:text-3xl font-semibold text-gray-800 dark:text-gray-100 mb-6 text-center transition-colors duration-300">
    Descubre lo que puedes monitorear
  </h2>
</div>
<!-- 🔹 Funcionalidades (cards) -->
<div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6 mt-10 px-6 md:px-12">
  <div class="bg-white dark:bg-gray-800 rounded-xl shadow-md p-6 text-center">
    <h3 class="text-lg font-semibold mb-2 text-gray-800 dark:text-gray-100">Monitoreo en tiempo real</h3>
    <p class="text-gray-600 dark:text-gray-300">Visualiza niveles de agua y humedad actualizados cada segundo.</p>
  </div>
  <div class="bg-white dark:bg-gray-800 rounded-xl shadow-md p-6 text-center">
    <h3 class="text-lg font-semibold mb-2 text-gray-800 dark:text-gray-100">Alertas automáticas</h3>
    <p class="text-gray-600 dark:text-gray-300">Recibe alertas de nivel crítico o condiciones de riesgo en tu zona.</p>
  </div>
  <div class="bg-white dark:bg-gray-800 rounded-xl shadow-md p-6 text-center">
    <h3 class="text-lg font-semibold mb-2 text-gray-800 dark:text-gray-100">Historial de datos</h3>
    <p class="text-gray-600 dark:text-gray-300">Consulta registros históricos y analiza tendencias de inundación.</p>
  </div>
</div>

<!-- 🔹 Segunda sección (texto izquierda, imagen derecha) -->
<div class="flex flex-col md:flex-row items-center justify-center gap-10 mt-10 px-6 md:px-12 transition-colors duration-300">

	<!-- Texto -->
	<div class="text-center md:text-left md:w-[45%] px-2 md:px-0">
		<h2 class="text-2xl md:text-3xl font-semibold mb-4 text-gray-800 dark:text-gray-100 transition-colors duration-300">
			¿Cómo lo hacemos?
		</h2>

		<p class="text-gray-700 dark:text-gray-300 text-base md:text-lg leading-relaxed transition-colors duration-300">
			Nuestras estaciones están equipadas con sensores de nivel de agua y humedad, que envían datos en tiempo real
			a nuestro sistema. Analizamos esta información automáticamente para generar alertas y mantenerte informado de
			cualquier riesgo potencial en tu zona.
		</p>
	</div>

	<!-- Imagen -->
	<img
		src={secundaria}
		alt="Imagen secundaria"
		class="
			rounded-2xl shadow-lg
			w-full max-w-xs           
			md:max-w-md md:w-[40%]   
			object-cover
		"
	/>
</div>
