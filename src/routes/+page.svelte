<script>
	import portada from "$lib/assets/img2.png";
</script>

<div class="flex flex-col md:flex-row items-center justify-center gap-10 mt-10 px-6 md:px-12 transition-colors duration-300">

	<!-- Imagen -->
	<img
		src={portada}
		alt="Imagen principal"
		class="
			rounded-2xl shadow-lg
			w-full max-w-xs            /* Móvil: imagen más chica y controlada */
			md:max-w-md md:w-[40%]    /* Escritorio */
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
