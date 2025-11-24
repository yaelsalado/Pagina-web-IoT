<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';
	import { ModeWatcher } from "mode-watcher";
	import LightSwitch from '$lib/components/light-switch/light-switch.svelte';
	import { writable } from "svelte/store";

	import CalendarIcon from "@lucide/svelte/icons/calendar";
	import HouseIcon from "@lucide/svelte/icons/house";
	import MapPinIcon from "@lucide/svelte/icons/map-pin";
	import DatabaseIcon from "@lucide/svelte/icons/database";
	import ServerIcon from "@lucide/svelte/icons/server";
	import MenuIcon from "@lucide/svelte/icons/menu";

	let { children } = $props();
	const isOpen = writable(false); // abierto/cerrado para móvil y escritorio

	const items = [
		{ title: "Inicio", url: "/", icon: HouseIcon },
		{ title: "Ubicaciones", url: "/ubicaciones", icon: MapPinIcon },
		{ title: "Datos", url: "/datos", icon: DatabaseIcon },
		{ title: "Estaciones", url: "/estaciones", icon: ServerIcon },
	];

	function toggleSidebar() {
		isOpen.update(v => !v);
	}
</script>

<svelte:head>
	<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet" />
	<link rel="icon" href={favicon} />
</svelte:head>

<style>
	:global(html) { font-family: 'Inter', sans-serif; }
</style>

<div class="flex h-screen bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">

	<!-- SIDEBAR ESCRITORIO -->
	<div
		class="h-full border-r transition-all duration-300 overflow-hidden
			   bg-gray-50 dark:bg-gray-800 border-gray-200 dark:border-gray-700
			   hidden md:flex flex-col"
		class:w-64={$isOpen}
		class:w-16={!$isOpen}
	>
		<!-- abrir/cerrar sidebar -->
		<button
			class="p-2 m-2 rounded cursor-pointer flex items-center justify-center
				   bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 dark:hover:bg-gray-600 transition-colors"
			on:click={toggleSidebar}
		>
			<MenuIcon size={22} />
		</button>

		<!-- Items -->
		<nav class="p-4 space-y-2">
			{#each items as item (item.title)}
				<a
					href={item.url}
					title={item.title}
					class="flex items-center gap-3 p-2 rounded transition-colors hover:bg-gray-200 dark:hover:bg-gray-700 text-gray-700 dark:text-gray-200"
				>
					<item.icon size={22} />
					<span class={`transition-opacity duration-300 ${$isOpen ? "opacity-100" : "opacity-0"}`}>
						{item.title}
					</span>
				</a>
			{/each}
		</nav>
	</div>

	<!-- MAIN AREA -->
	<div class="flex-1 flex flex-col">

		<!-- TOP BAR -->
		<div class="w-full bg-gray-200 dark:bg-gray-800 text-gray-700 dark:text-gray-200 py-3 px-6
		            flex justify-between items-center">

			<!-- Botón móvil -->
			<div class="md:hidden">
				<button
					class="p-2 rounded bg-gray-300 dark:bg-gray-700"
					on:click={toggleSidebar}
				>
					<MenuIcon size={24} />
				</button>
			</div>

			<!-- Título -->
			<div class="flex flex-col text-lg md:text-xl font-semibold tracking-wide ml-4 md:ml-0">
				<span>TeCuidamos 💧</span>
			</div>

			<!-- Modo oscuro -->
			<LightSwitch />
		</div>

		<!-- SIDEBAR MÓVIL -->
		{#if $isOpen}
			<!-- Fondo oscuro -->
			<div class="absolute inset-0 bg-black/40 backdrop-blur-sm md:hidden"
				on:click={toggleSidebar}></div>

			<!-- Drawer -->
			<div class="absolute left-0 top-0 h-full w-64 bg-gray-50 dark:bg-gray-800 shadow-xl md:hidden z-20 p-4 space-y-4">
				{#each items as item (item.title)}
					<a href={item.url}
						class="flex items-center gap-3 p-2 rounded hover:bg-gray-200 dark:hover:bg-gray-700">
						<item.icon size={22} />
						{item.title}
					</a>
				{/each}
			</div>
		{/if}

		<!-- PAGE CONTENT -->
		<main class="flex-1 p-4 md:p-6 overflow-auto">
			<div class="bg-white dark:bg-gray-800 p-4 md:p-6 rounded-xl shadow">
				{@render children?.()}
			</div>
		</main>

		<!-- FOOTER -->
		<footer class="text-center text-gray-400 dark:text-gray-500 text-sm py-4">
			© 2025 TeCuidamos
		</footer>

	</div>
</div>

<ModeWatcher />
