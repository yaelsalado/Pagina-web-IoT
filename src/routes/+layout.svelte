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
	const isOpen = writable(true);

	const items = [
		{ title: "Inicio", url: "/", icon: HouseIcon },
		{ title: "Ubicaciones", url: "/ubicaciones", icon: MapPinIcon },
		{ title: "Datos", url: "/datos", icon: DatabaseIcon },
		{ title: "Estaciones", url: "/estaciones", icon: ServerIcon },
	];
	
	function toggleSidebar() {
		isOpen.update((v) => !v);
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

	<!-- Sidebar -->
	<div
		class="h-full border-r transition-all duration-300 overflow-hidden
			   bg-gray-50 dark:bg-gray-800 border-gray-200 dark:border-gray-700"
		class:w-64={$isOpen}
		class:w-16={!$isOpen}
	>
		<!-- Trigger -->
		<button
			class="p-2 m-2 rounded cursor-pointer flex items-center justify-center
				   bg-gray-200 dark:bg-gray-700 hover:bg-gray-300 dark:hover:bg-gray-600 transition-colors"
			on:click={toggleSidebar}
		>
			<MenuIcon size={22} />
		</button>

		<!-- Menu items -->
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

	<!-- Main content -->
	<div class="flex-1 flex flex-col">
		
		<!-- Top bar -->
		<div class="w-full bg-gray-200 dark:bg-gray-800 text-gray-700 dark:text-gray-200 py-3 px-6 text-left text-lg font-semibold tracking-wide flex justify-between items-center">
			<div class="flex flex-col">
				<span>TeCuidamos 💧</span>
			</div>
			<LightSwitch />
		</div>

		<!-- Page content -->
		<main class="flex-1 p-6 overflow-auto">
			<div class="bg-white dark:bg-gray-800 p-6 rounded-xl shadow">
				{@render children?.()}
			</div>
		</main>

		<!-- Footer discreto -->
		<footer class="text-center text-gray-400 dark:text-gray-500 text-sm py-4">
			© 2025 TeCuidamos
		</footer>
	</div>

</div>

<ModeWatcher />
