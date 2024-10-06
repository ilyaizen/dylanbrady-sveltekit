<script lang="ts">
  import { PrismicPreview } from '@prismicio/svelte/kit';
  import { repositoryName } from '$lib/prismicio';
  import { page } from '$app/stores';
  import { goto } from '$app/navigation';
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import '@fontsource-variable/grandstander';
  import '@fontsource/abril-fatface';
  import '@fontsource-variable/montserrat';
  import '@fontsource/roboto';
  import '$lib/styles/hebrew-font.css';
  import '../app.css';
  import Loader from '$lib/components/Loader.svelte';
  import { loading } from '$lib/stores/loading';
  import { darkMode } from '$lib/stores/darkMode';
  import { gsap } from 'gsap';
  import { Button } from '$lib/components/ui/button';
  import { Moon, Sun, RotateCcw } from 'lucide-svelte';
  import GradientCanvas from '$lib/components/GradientCanvas.svelte';

  // Lucide Svelte
  import { Home, PencilLine, TvMinimalPlay } from 'lucide-svelte';
  // Simple Icons : simpleicons.org
  // import TwitterSvg from '$lib/svg/web/x.svg';
  // import GithubSvg from '$lib/svg/web/github.svg';
  // import LinkedInSvg from '$lib/svg/web/linkedin.svg';
  // import MailSvg from '$lib/svg/web/gmail.svg';
  //    Shadcn Components
  import * as Tooltip from '$lib/components/ui/tooltip';
  import Separator from '$lib/components/ui/separator/separator.svelte';
  //   Major Components
  import Dock from '$lib/components/Dock.svelte';
  import DockIcon from '$lib/components/DockIcon.svelte';

  let navs = {
    navbar: [
      { label: 'Home', icon: Home, href: '/', onClick: () => goto('/') },
      { label: 'Blog', icon: PencilLine, href: '/blog', onClick: () => goto('/blog') },
      { label: 'Developer', icon: TvMinimalPlay, href: '/developer', onClick: () => goto('/developer') }
    ]
    // contact: [
    //   { label: 'Github', icon: GithubSvg, href: '#' },
    //   { label: 'LinkedIn', icon: LinkedInSvg, href: '#' },
    //   { label: 'X', icon: TwitterSvg, href: '#' },
    //   { label: 'Email', icon: MailSvg, href: '#' }
    // ]
  };

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
      animateThemeChange($darkMode ? '#000000' : '#ffffff', $darkMode ? '#ffffff' : '#000000');
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

  function handleNavClick(href: string) {
    console.log('Navigation clicked:', href);
    goto(href);
  }

  function handleLanguageToggle() {
    console.log('Language toggle clicked');
    toggleLanguage();
  }

  function handleRefresh() {
    console.log('Refresh clicked');
    window.location.reload();
  }

  function handleDarkModeToggle() {
    console.log('Dark mode toggle clicked');
    toggleDarkMode();
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

<GradientCanvas />

<div dir={isRTL ? 'rtl' : 'ltr'} class="px-4 md:px-6 mx-auto space-y-8 w-full max-w-5xl relative z-10">
  {#if $loading}
    <Loader onLoadComplete={handleLoadComplete} />
  {:else}
    <main style="font-family: {isRTL ? 'Almoni' : 'Almoni'}, system-ui;">
      <slot />
    </main>
    <PrismicPreview {repositoryName} />
  {/if}
</div>

<Dock
  direction="middle"
  class="fixed bottom-4 left-1/2 transform -translate-x-1/2 z-50"
  let:mouseX
  let:distance
  let:magnification
>
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
  <DockIcon {mouseX} {magnification} {distance} on:click={handleLanguageToggle}>
    <Tooltip.Root>
      <Tooltip.Trigger class="hover:bg-zinc-900/80 transition-all duration-200 rounded-full p-3 mx-0">
        {$lang === 'en-us' ? 'עב' : 'En'}
      </Tooltip.Trigger>
      <Tooltip.Content sideOffset={8}>
        <p>Toggle language</p>
      </Tooltip.Content>
    </Tooltip.Root>
  </DockIcon>
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
  <DockIcon {mouseX} {magnification} {distance} on:click={handleDarkModeToggle}>
    <Tooltip.Root>
      <Tooltip.Trigger class="hover:bg-zinc-900/80 transition-all duration-200 rounded-full p-3 mx-0">
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
