<script lang="ts">
	import { loading } from '$lib/stores/loading';
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';

	export let onLoadComplete: () => void;

	let loadingElement: HTMLElement;

	onMount(async () => {
		// Wait for all resources to load
		await Promise.all([waitForResources(), waitForFonts(), waitForImages()]);

		// Add a small delay to ensure smooth transition
		await new Promise((resolve) => setTimeout(resolve, 500));

		// Animate out the loading screen
		await gsap.to(loadingElement, {
			opacity: 0,
			duration: 0.5,
			ease: 'expo.out'
		});

		loading.set(false);
		onLoadComplete();
	});

	// Functions to wait for various resources to load
	function waitForResources() {
		return new Promise((resolve) => {
			if (document.readyState === 'complete') {
				resolve(null);
			} else {
				window.addEventListener('load', () => resolve(null));
			}
		});
	}

	function waitForFonts() {
		return document.fonts.ready;
	}

	function waitForImages() {
		const images = Array.from(document.images);
		const imagePromises = images.map((img) => {
			if (img.complete) {
				return Promise.resolve();
			} else {
				return new Promise((resolve) => {
					img.onload = img.onerror = () => resolve(null);
				});
			}
		});
		return Promise.all(imagePromises);
	}
</script>

<div class="loading-screen" bind:this={loadingElement}>
	<div class="color-wheel"></div>
</div>

<style>
	.loading-screen {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		display: flex;
		justify-content: flex-end;
		align-items: flex-start;
		z-index: 9999;
		opacity: 1;
	}

	.color-wheel {
		width: 300px;
		height: 300px;
		border-radius: 100%;
		background: conic-gradient(in hsl longer hue, red 0 0);
		animation: spin 0.3s linear infinite;
		transform: translate(-50%, -50%) scale(15);
		position: absolute;
		top: 0;
		left: 50%;
	}

	@keyframes spin {
		0% {
			transform: translate(-50%, -50%) scale(15) rotate(0deg);
		}
		100% {
			transform: translate(-50%, -50%) scale(15) rotate(360deg);
		}
	}
</style>
