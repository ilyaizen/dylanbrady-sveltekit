<script lang="ts">
  // Import necessary Svelte and SvelteKit modules
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import { goto } from '$app/navigation';
  import { writable } from 'svelte/store';
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';

  // Import Prismic preview component
  import { PrismicPreview } from '@prismicio/svelte/kit';
  import { repositoryName } from '$lib/prismicio';

  // Import GSAP for animations
  import { gsap } from 'gsap';

  // Import custom stores
  import { loading } from '$lib/stores/loading';
  import { darkMode } from '$lib/stores/darkMode';

  // Import UI components and icons
  import { Button } from '$lib/components/ui/button';
  import { Moon, Sun, RotateCcw, Home, PencilLine, TvMinimalPlay } from 'lucide-svelte';
  import * as Tooltip from '$lib/components/ui/tooltip';
  import Separator from '$lib/components/ui/separator/separator.svelte';

  // Import custom components
  import Loader from '$lib/components/Loader.svelte';
  import GradientCanvas from '$lib/components/GradientCanvas.svelte';
  import Dock from '$lib/components/Dock.svelte';
  import DockIcon from '$lib/components/DockIcon.svelte';

  // Import styles
  import '@fontsource-variable/grandstander';
  import '@fontsource/abril-fatface';
  import '@fontsource-variable/montserrat';
  import '@fontsource/roboto';
  import '$lib/styles/hebrew-font.css';
  import '../app.css';

  // Define navigation items
  const navs = {
    navbar: [
      { label: 'Home', icon: Home, href: '/', onClick: () => goto('/') },
      { label: 'Blog', icon: PencilLine, href: '/blog', onClick: () => goto('/blog') },
      { label: 'Developer', icon: TvMinimalPlay, href: '/developer', onClick: () => goto('/developer') }
    ]
  };

  // Animation for pop-up effect
  function popUpAnimation(node: HTMLElement) {
    gsap.from(node, {
      y: -100,
      opacity: 0,
      duration: 0.5,
      ease: 'back.out(1.7)'
    });

    return {
      destroy() {
        // Clean up if needed
      }
    };
  }

  // Store for the current language
  const lang = writable('en-us');

  // Determine if the current language is right-to-left
  $: isRTL = $lang === 'he';

  // Store for page load status
  const pageLoaded = writable(false);

  let lastScrollY = 0;
  const dockPosition = tweened(0, {
    duration: 300,
    easing: cubicOut
  });

  onMount(() => {
    // Update language based on page data
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
      animateThemeChange($darkMode ? '#000000' : '#ffffff', $darkMode ? '#ffffff' : '#000000');
      localStorage.setItem('darkMode', JSON.stringify($darkMode));
    });

    // Set pageLoaded to true after a short delay
    setTimeout(() => {
      pageLoaded.set(true);
    }, 500);

    const handleScroll = () => {
      const currentScrollY = window.scrollY;
      if (Math.abs(currentScrollY - lastScrollY) > 50) {
        if (currentScrollY > lastScrollY) {
          // Scrolling down
          dockPosition.set(-100);
        } else {
          // Scrolling up
          dockPosition.set(0);
        }
        lastScrollY = currentScrollY;
      }
    };

    window.addEventListener('scroll', handleScroll);

    // Cleanup subscriptions on component unmount
    return () => {
      unsubscribe();
      unsubscribeDarkMode();
      window.removeEventListener('scroll', handleScroll);
    };
  });

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

  // Handler functions for dock icons
  function handleNavClick(href: string) {
    goto(href);
  }

  function handleLanguageToggle() {
    toggleLanguage();
  }

  function handleRefresh() {
    window.location.reload();
  }

  function handleDarkModeToggle() {
    toggleDarkMode();
  }
</script>

<!-- Add meta tags for SEO -->
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

<!-- Gradient background -->
<GradientCanvas />

<!-- Main content container -->
<div dir={isRTL ? 'rtl' : 'ltr'} class="px-4 md:px-6 mx-auto space-y-8 w-full max-w-3xl relative z-10">
  {#if $loading}
    <Loader onLoadComplete={() => ($loading = false)} />
  {:else}
    <main style="font-family: {isRTL ? 'Almoni' : 'Almoni'}, system-ui;">
      <slot />
    </main>
    <PrismicPreview {repositoryName} />
  {/if}
</div>

<!-- Dock with navigation and control icons -->
{#if $pageLoaded}
  <div class="dock-container" style="transform: translateY({$dockPosition}px);">
    <Dock
      direction="middle"
      class="fixed top-0 left-1/2 -translate-x-1/2 z-50 rounded-full bg-white/20 dark:bg-black/20"
      let:mouseX
      let:distance
      let:magnification
    >
      <!-- Navigation icons -->
      {#each navs.navbar as item}
        <DockIcon {mouseX} {magnification} {distance} on:click={() => handleNavClick(item.href)}>
          <Tooltip.Root>
            <Tooltip.Trigger class="hover:bg-zinc-900/80 transition-all duration-200 rounded-full p-3 mx-0">
              <svelte:component this={item.icon} size={22} strokeWidth={1.2} />
            </Tooltip.Trigger>
            <Tooltip.Content sideOffset={8}>
              <p>{item.label}</p>
            </Tooltip.Content>
          </Tooltip.Root>
        </DockIcon>
      {/each}

      <Separator orientation="vertical" class="h-full w-[0.6px]" />

      <!-- Language toggle -->
      <DockIcon {mouseX} {magnification} {distance} on:click={handleLanguageToggle}>
        <Tooltip.Root>
          <Tooltip.Trigger
            class="language-toggle hover:bg-zinc-900/80 transition-all duration-200 rounded-full p-3 mx-0 flex items-center justify-center"
          >
            <span class="text-sm font-medium">{$lang === 'en-us' ? 'עב' : 'En'}</span>
          </Tooltip.Trigger>
          <Tooltip.Content sideOffset={8}>
            <p>Toggle language</p>
          </Tooltip.Content>
        </Tooltip.Root>
      </DockIcon>

      <!-- Refresh button -->
      <DockIcon {mouseX} {magnification} {distance} on:click={handleRefresh}>
        <Tooltip.Root>
          <Tooltip.Trigger class="hover:bg-zinc-900/80 transition-all duration-200 rounded-full p-3 mx-0">
            <RotateCcw size={22} strokeWidth={1.2} />
          </Tooltip.Trigger>
          <Tooltip.Content sideOffset={8}>
            <p>Refresh page</p>
          </Tooltip.Content>
        </Tooltip.Root>
      </DockIcon>

      <!-- Dark mode toggle -->
      <DockIcon {mouseX} {magnification} {distance} on:click={handleDarkModeToggle}>
        <Tooltip.Root>
          <Tooltip.Trigger class="hover:bg-zinc-900/80 rounded-full p-3 mx-0">
            {#if $darkMode}
              <Sun size={22} strokeWidth={1.2} />
            {:else}
              <Moon size={22} strokeWidth={1.2} />
            {/if}
          </Tooltip.Trigger>
          <Tooltip.Content sideOffset={8}>
            <p>{$darkMode ? 'Light mode' : 'Dark mode'}</p>
          </Tooltip.Content>
        </Tooltip.Root>
      </DockIcon>
    </Dock>
  </div>
{/if}

<style>
  /* Dock container positioning */
  .dock-container {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 50;
    transition: transform 0.3s ease-out;
  }
</style>
