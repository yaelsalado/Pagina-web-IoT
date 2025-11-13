<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';
	import * as NavigationMenu from "$lib/components/ui/navigation-menu/index.js";
	import { navigationMenuTriggerStyle } from "$lib/components/ui/navigation-menu/navigation-menu-trigger.svelte";
	import { ModeWatcher } from "mode-watcher";
	import LightSwitch from '$lib/components/light-switch/light-switch.svelte';
	let { children } = $props();
</script>

<svelte:head>
	<!-- Importa la fuente -->
	<link
		href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
		rel="stylesheet"
	/>
	<link rel="icon" href={favicon} />
</svelte:head>

<style>
	:global(html) {
		font-family: 'Inter', sans-serif;
	}
</style>

<!-- Header con nombre del proyecto -->
<div
	class="w-full bg-gray-200 dark:bg-gray-800 text-gray-700 dark:text-gray-200 py-2 px-6 text-left text-lg font-semibold tracking-wide"
>
	TeCuidamos 💧
</div>

<!-- Barra de navegación -->
<div class="relative px-6">
	<NavigationMenu.Root viewport={false}>
		<NavigationMenu.List>
			<NavigationMenu.Item>
				<NavigationMenu.Link>
					{#snippet child()}
						<a href="/" class={navigationMenuTriggerStyle()}>Inicio</a>
					{/snippet}
				</NavigationMenu.Link>
			</NavigationMenu.Item>

			<NavigationMenu.Item>
				<NavigationMenu.Link>
					{#snippet child()}
						<a href="/ubicaciones" class={navigationMenuTriggerStyle()}>Ubicaciones</a>
					{/snippet}
				</NavigationMenu.Link>
			</NavigationMenu.Item>

			<NavigationMenu.Item>
				<NavigationMenu.Link>
					{#snippet child()}
						<a href="/datos" class={navigationMenuTriggerStyle()}>Datos</a>
					{/snippet}
				</NavigationMenu.Link>
			</NavigationMenu.Item>

			<NavigationMenu.Item>
				<NavigationMenu.Link>
					{#snippet child()}
						<a href="/tec" class={navigationMenuTriggerStyle()}>Estación Tec</a>
					{/snippet}
				</NavigationMenu.Link>
			</NavigationMenu.Item>

			<NavigationMenu.Item>
				<NavigationMenu.Link>
					{#snippet child()}
						<a href="/chapalita" class={navigationMenuTriggerStyle()}>Estación Chapalita</a>
					{/snippet}
				</NavigationMenu.Link>
			</NavigationMenu.Item>

			<!-- 🆕 Nueva pestaña -->
			<NavigationMenu.Item>
				<NavigationMenu.Link>
					{#snippet child()}
						<a href="/americana" class={navigationMenuTriggerStyle()}>Estación Americana</a>
					{/snippet}
				</NavigationMenu.Link>
			</NavigationMenu.Item>
		</NavigationMenu.List>
	</NavigationMenu.Root>

	<!-- LightSwitch -->
	<div class="absolute top-3 right-6">
		<LightSwitch />
	</div>
</div>

<ModeWatcher />
{@render children?.()}
