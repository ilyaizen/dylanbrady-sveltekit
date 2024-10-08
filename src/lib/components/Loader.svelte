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
  <div class="loader"></div>
</div>

<style>
  .loading-screen {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 9999;
    opacity: 1;
    background-color: rgba(255, 255, 255, 0.9);
  }

  .loader {
    width: 50px;
    height: 50px;
    border: 3px solid #f3f3f3;
    border-top: 3px solid #3498db;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
</style>
