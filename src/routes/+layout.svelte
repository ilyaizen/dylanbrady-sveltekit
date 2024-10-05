<script lang="ts">
	import { PrismicPreview } from '@prismicio/svelte/kit';
	import { repositoryName } from '$lib/prismicio';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	import '@fontsource-variable/grandstander';
	import '$lib/styles/hebrew-font.css';
	import '../app.css';
	import { loading } from '$lib/stores/loading';
	import { darkMode } from '$lib/stores/darkMode';
	import { gsap } from 'gsap';
	import { Button } from '$lib/components/ui/button';
	import { Moon, Sun, RotateCcw } from 'lucide-svelte';
	import Loader from '$lib/components/Loader.svelte';

	// Store for the current language
	const lang = writable('en-us');

	// Determine if the current language is right-to-left
	$: isRTL = $lang === 'he';

	let resourcesLoaded = false;

	onMount(() => {
		// Subscribe to page changes to update the language
		const unsubscribe = page.subscribe(($page) => {
			if ($page.data) {
				lang.set($page.params.lang || 'en-us');
			}
		});

		// Initialize dark mode from localStorage
		const savedDarkMode = localStorage.getItem('darkMode');
		if (savedDarkMode) {
			darkMode.set(JSON.parse(savedDarkMode));
		}

		// Update body class and animate when dark mode changes
		const unsubscribeDarkMode = darkMode.subscribe(($darkMode) => {
			document.body.classList.toggle('dark', $darkMode);
			animateThemeChange($darkMode ? '#1a1a1a' : '#ffffff', $darkMode ? '#ffffff' : '#000000');
			localStorage.setItem('darkMode', JSON.stringify($darkMode));
		});

		// Cleanup subscriptions on component unmount
		return () => {
			unsubscribe();
			unsubscribeDarkMode();
		};
	});

	function handleLoadComplete() {
		resourcesLoaded = true;
	}

	// Toggle dark mode
	function toggleDarkMode() {
		darkMode.update((value) => !value);
	}

	// Animate theme change
	function animateThemeChange(bgColor: string, textColor: string) {
		gsap.to(document.body, {
			backgroundColor: bgColor,
			color: textColor,
			duration: 0.3,
			ease: 'expo.out'
		});
	}

	// Toggle language between English and Hebrew
	function toggleLanguage() {
		const newLang = $lang === 'en-us' ? 'he' : 'en-us';
		const newPath = newLang === 'en-us' ? '/' : '/he/';
		goto(newPath + $page.url.pathname.split('/').slice(2).join('/'));
	}
</script>

<svelte:head>
	<title>{$page.data.title}</title>
	{#if $page.data.meta_description}
		<meta name="description" content={$page.data.meta_description} />
	{/if}
	{#if $page.data.meta_title}
		<meta name="og:title" content={$page.data.meta_title} />
	{/if}
	{#if $page.data.meta_image}
		<meta name="og:image" content={$page.data.meta_image} />
		<meta name="twitter:card" content="summary_large_image" />
	{/if}
</svelte:head>

<div dir={isRTL ? 'rtl' : 'ltr'} class="px-4 md:px-6 mx-auto space-y-8 w-full max-w-2xl">
	{#if $loading}
		<Loader onLoadComplete={handleLoadComplete} />
	{:else}
		<div class="fixed bottom-4 right-4 flex gap-2 z-50">
			<Button variant="ghost" size="icon" on:click={toggleLanguage}>
				{$lang === 'en-us' ? 'עב' : 'En'}
				<span class="sr-only">Toggle language</span>
			</Button>
			<Button variant="ghost" size="icon" on:click={() => window.location.reload()}>
				<RotateCcw class="h-[1.2rem] w-[1.2rem]" />
				<span class="sr-only">Refresh page</span>
			</Button>
			<Button variant="ghost" size="icon" on:click={toggleDarkMode}>
				{#if $darkMode}
					<Sun class="h-[1.2rem] w-[1.2rem]" />
					<span class="sr-only">Light mode</span>
				{:else}
					<Moon class="h-[1.2rem] w-[1.2rem]" />
					<span class="sr-only">Dark mode</span>
				{/if}
			</Button>
		</div>
		<main style="font-family: {isRTL ? 'YardenAlefAlefAlef' : 'Grandstander Variable'}, system-ui;">
			<slot />
		</main>
		<PrismicPreview {repositoryName} />
	{/if}
</div>

<style>
	:global(body) {
		transition:
			background-color 0.5s,
			color 0.5s;
		background-color: transparent;
	}

	:global(body.dark) {
		color: #ffffff;
	}
</style>
