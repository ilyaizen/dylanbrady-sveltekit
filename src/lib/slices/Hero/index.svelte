<script lang="ts">
  import { onMount } from 'svelte';
  import type { Content } from '@prismicio/client';
  import gsap from 'gsap';
  import { PrismicImage } from '@prismicio/svelte';
  import { page } from '$app/stores';

  export let slice: Content.HeroSlice;

  $: isHebrew = $page.params.lang === 'he';

  $: greetings = isHebrew ? ['שלום', 'שלום,'] : ['Hey', 'there,'];

  onMount(() => {
    const tl = gsap.timeline({
      onComplete: () => {
        // Dispatch a custom event when the hero animation completes
        window.dispatchEvent(new CustomEvent('heroAnimationComplete'));
      }
    });

    tl.fromTo(
      '.welcome-animation-word',
      {
        opacity: 0
      },
      {
        delay: 0.5,
        opacity: 1,
        duration: 0.001,
        stagger: 0.1 // Reduced from 0.2 to 0.1 for quicker animation
      }
    )
      .fromTo(
        '.wave-animation',
        {
          x: -10,
          opacity: 0
        },
        {
          x: 0,
          opacity: 1,
          ease: 'expo.out',
          duration: 0.4 // Reduced from 0.8 to 0.4
        }
      )
      .fromTo(
        '.welcome-animation-name',
        {
          x: -10,
          opacity: 0
        },
        {
          x: 0,
          ease: 'expo.out',
          opacity: 1,
          stagger: 0.2 // Delay between each word
        }
      )
      .fromTo(
        '.profile-image-animation',
        {
          x: -20,
          opacity: 0
        },
        {
          x: 0,
          opacity: 1,
          ease: 'expo.out',
          duration: 1.2
        },
        0 // Start at the same time as the welcome animation
      )
      .fromTo(
        '.tagline-animation',
        {
          x: -10,
          opacity: 0
        },
        {
          x: 0,
          opacity: 1,
          ease: 'expo.out',
          duration: 0.8,
          stagger: 0.025
        },
        '-=0.6'
      )
      .fromTo(
        '.about-heading',
        {
          x: -5,
          opacity: 0
        },
        {
          x: 0,
          opacity: 1,
          duration: 0.8,
          delay: 0.5,
          ease: 'expo.out'
        },
        '-=1.5'
      )
      .fromTo(
        '.about-text-animation',
        {
          x: -5,
          opacity: 0
        },
        {
          x: 0,
          opacity: 1,
          duration: 0.4,
          stagger: 0.01,
          ease: 'expo.out'
        },
        '-=0.8'
      );

    // Add jiggle and click animations for wave emoji
    const waveEmoji = document.querySelector('.wave-animation');
    if (waveEmoji) {
      // Jiggle animation on hover
      waveEmoji.addEventListener('mouseenter', () => {
        gsap.to(waveEmoji, {
          rotation: 20,
          scale: 1.2,
          duration: 0.3,
          ease: 'elastic.out(1, 0.3)',
          onComplete: () => {
            gsap.to(waveEmoji, {
              rotation: -15,
              yoyo: true,
              repeat: 5,
              duration: 0.1,
              ease: 'power1.inOut'
            });
          }
        });
      });

      waveEmoji.addEventListener('mouseleave', () => {
        gsap.to(waveEmoji, {
          rotation: 0,
          scale: 1,
          duration: 0.5,
          ease: 'elastic.out(1, 0.3)'
        });
      });

      // Special animation on click
      waveEmoji.addEventListener('click', () => {
        gsap.to(waveEmoji, {
          rotation: 360,
          scale: 1.5,
          y: -20,
          duration: 0.5,
          ease: 'back.out(1.7)',
          onComplete: () => {
            gsap.to(waveEmoji, {
              rotation: 0,
              scale: 1,
              y: 0,
              duration: 0.5,
              ease: 'bounce.out'
            });
          }
        });
      });
    }
  });
</script>

<section data-slice-type={slice.slice_type} data-slice-variation={slice.variation} id="hero">
  <div class="pt-40 flex flex-col space-y-4">
    <div class="flex flex-wrap items-center">
      {#each greetings as word}
        <span class="welcome-animation-word inline-block text-5xl sm:text-7xl font-black tracking-tighter">
          {word}&nbsp;
        </span>
      {/each}
      <span class="wave-animation inline-block text-5xl sm:text-7xl font-black tracking-tighter cursor-pointer ml-2"
        >👋</span
      >
    </div>

    <div class="flex flex-wrap items-center">
      <span class="welcome-animation-word inline-block text-5xl sm:text-7xl font-black tracking-tighter">
        {isHebrew ? 'אני' : "I'm"}&nbsp;
      </span>
      {#each [slice.primary.first_name, slice.primary.last_name] as word}
        <span class="welcome-animation-name inline-block text-6xl sm:text-8xl font-black tracking-tighter">
          {word}&nbsp;
        </span>
      {/each}
    </div>

    {#if slice.primary.tag_line}
      <div class="flex flex-wrap">
        {#each slice.primary.tag_line.split(' ') as word}
          <span class="tagline-animation inline-block text-sm sm:text-xl">{word}&nbsp;</span>
        {/each}
      </div>
    {/if}
  </div>
</section>
