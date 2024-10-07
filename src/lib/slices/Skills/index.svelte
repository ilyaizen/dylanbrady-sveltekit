<script lang="ts">
  import type { Content } from '@prismicio/client';
  import { onMount } from 'svelte';
  import gsap from 'gsap';

  export let slice: Content.SkillsSlice;

  let sectionRef: HTMLElement;

  onMount(() => {
    const tl = gsap.timeline({ paused: true });

    tl.fromTo(
      '.skills-heading-animation',
      { y: 20, opacity: 0 },
      { y: 0, opacity: 1, duration: 0.3, ease: 'expo.out' }
    ).fromTo(
      '.skills-item-animation',
      { y: 20, opacity: 0 },
      { y: 0, opacity: 1, duration: 0.4, stagger: 0.03, ease: 'expo.out' },
      '-=0.2'
    );

    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            tl.play();
            observer.unobserve(entry.target);
          }
        });
      },
      { threshold: 0.2 }
    );

    if (sectionRef) {
      observer.observe(sectionRef);
    }

    return () => {
      if (sectionRef) {
        observer.unobserve(sectionRef);
      }
    };
  });
</script>

<section
  bind:this={sectionRef}
  class="max-w-xl mx-auto font-roboto"
  id="skills"
  data-slice-type={slice.slice_type}
  data-slice-variation={slice.variation}
>
  <h2 class="skills-heading-animation text-xl font-bold pt-4 pb-4">
    {slice.primary.skills_heading}
  </h2>
  <div class="flex flex-wrap gap-1">
    {#each slice.primary.skills_item as item}
      <div
        class="skills-item-animation inline-flex items-center rounded-md border px-2.5 py-0.5 text-xs font-semibold transition-colors focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 border-transparent bg-primary text-primary-foreground shadow hover:bg-primary/80"
      >
        {item.skill}
      </div>
    {/each}
  </div>
</section>
