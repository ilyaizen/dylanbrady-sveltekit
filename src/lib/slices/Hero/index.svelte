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
      )
      .to(
        '.welcome-animation-name',
        {
          textShadow: `
            1px 1px 1px rgba(160, 160, 160, 0.3),
            2px 2px 2px rgba(160, 160, 160, 0.25),
            3px 3px 3px rgba(160, 160, 160, 0.2),
            5px 5px 4px rgba(160, 160, 160, 0.15),
            8px 8px 5px rgba(160, 160, 160, 0.1),
            12px 12px 6px rgba(160, 160, 160, 0.08),
            17px 17px 7px rgba(160, 160, 160, 0.06),
            23px 23px 8px rgba(160, 160, 160, 0.05),
            30px 30px 9px rgba(160, 160, 160, 0.04),
            38px 38px 10px rgba(160, 160, 160, 0.03),
            47px 47px 11px rgba(160, 160, 160, 0.02),
            57px 57px 12px rgba(160, 160, 160, 0.015),
            68px 68px 13px rgba(160, 160, 160, 0.01),
            80px 80px 14px rgba(160, 160, 160, 0.005),
            93px 93px 15px rgba(160, 160, 160, 0.003)
          `,
          duration: 1,
          ease: 'expo.out'
        },
        '-=1'
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

<section
  data-slice-type={slice.slice_type}
  data-slice-variation={slice.variation}
  id="hero"
  class="container mx-auto px-4"
>
  <div class="pt-20 sm:pt-32 md:pt-40 flex flex-col md:flex-row items-center md:items-start justify-between">
    <div class="flex flex-col space-y-2 sm:space-y-4 text-center md:text-left">
      <div class="flex flex-wrap items-center justify-center md:justify-start">
        {#each greetings as word}
          <span
            class="welcome-animation-word inline-block text-3xl sm:text-4xl md:text-5xl font-extrabold tracking-tight"
          >
            {word}&nbsp;
          </span>
        {/each}
        <span
          class="wave-animation inline-block text-3xl sm:text-4xl md:text-5xl font-extrabold cursor-pointer ml-2 tracking-tight"
          >👋</span
        >
      </div>

      <div class="flex flex-wrap items-center justify-center md:justify-start">
        <span
          class="welcome-animation-word inline-block text-3xl sm:text-4xl md:text-5xl font-extrabold tracking-tight"
        >
          {isHebrew ? 'אני' : "I'm"}&nbsp;
        </span>
        {#each [slice.primary.first_name, slice.primary.last_name] as word}
          <span class="welcome-animation-name inline-block text-4xl sm:text-5xl md:text-6xl font-black">
            {word}&nbsp;
          </span>
        {/each}
      </div>

      {#if slice.primary.tag_line}
        <div class="flex flex-wrap font-roboto justify-center md:justify-start">
          {#each slice.primary.tag_line.split(' ') as word}
            <span class="tagline-animation inline-block text-sm sm:text-base md:text-lg">{word}&nbsp;</span>
          {/each}
        </div>
      {/if}
    </div>

    <span
      class="profile-image-animation flex overflow-hidden relative shrink-0 size-32 sm:size-40 md:size-48 rounded-full shadow-xl mt-6 md:mt-0 md:ml-4"
    >
      <div role="figure" class="aspect-square w-full h-full">
        <PrismicImage field={slice.primary.profile_picture} />
      </div>
    </span>
  </div>

  <section id="about" class="font-roboto mt-8 md:mt-12 max-w-xl mx-auto">
    <h2 class="text-lg sm:text-xl font-bold about-heading">{slice.primary.about_heading}</h2>
    <div class="max-w-full pt-2 sm:pt-4">
      {#if slice.primary.about_line}
        {#each slice.primary.about_line.split(' ') as word}
          <span class="about-text-animation inline-block">{word}&nbsp;</span>
        {/each}
      {/if}
    </div>
  </section>
</section>
