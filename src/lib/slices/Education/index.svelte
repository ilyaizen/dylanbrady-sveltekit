<script lang="ts">
  import type { Content } from '@prismicio/client';
  import { onMount } from 'svelte';
  import gsap from 'gsap';

  export let slice: Content.EducationSlice;

  let sectionRef: HTMLElement;
  let tl: gsap.core.Timeline;

  onMount(() => {
    tl = gsap.timeline({ paused: true });

    tl.fromTo(
      '.education-heading-animation',
      { x: -5, opacity: 0 },
      { x: 0, opacity: 1, duration: 0.3, ease: 'expo.out' }
    ).fromTo(
      '.education-item-animation',
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

  // Remove the startEducationAnimation function as it's no longer needed

  function getIconStyle(color: string | undefined, iconUrl: string) {
    return `
			background-color: ${color || '#000000'};
				-webkit-mask-image: url(${iconUrl});
				mask-image: url(${iconUrl});
				-webkit-mask-size: contain;
				mask-size: contain;
				-webkit-mask-repeat: no-repeat;
				mask-repeat: no-repeat;
				-webkit-mask-position: center;
				mask-position: center;
		`;
  }
</script>

<section
  bind:this={sectionRef}
  class="max-w-xl mx-auto font-roboto"
  id="education"
  data-slice-type={slice.slice_type}
  data-slice-variation={slice.variation}
>
  <h2 class="education-heading-animation text-xl font-bold pt-4 pb-4">
    {slice.primary.education_heading}
  </h2>
  <div>
    {#each slice.primary.education_item as item}
      <div class="education-item-animation">
        <a class="block cursor-pointer pb-4" href={item.education_url}>
          <div class="rounded-full bg-card/20 text-card-foreground flex items-center">
            <div class="flex-none">
              <span class="relative flex shrink-0 overflow-hidden size-12 m-auto">
                <div
                  class="w-full h-full"
                  style={getIconStyle(item.education_icon_color ?? '', item.education_icon?.url ?? '')}
                ></div>
              </span>
            </div>
            <div class="flex-grow ml-4 flex items-center justify-between">
              <div class="flex flex-col">
                <h3 class="inline-flex items-center justify-start font-semibold leading-none text-xs sm:text-sm group">
                  {item.education_title}
                </h3>
                <div class="font-sans text-xs">{item.education_description}</div>
              </div>
              <div class="text-xs sm:text-sm tabular-nums text-muted-foreground text-right ml-4 mr-4 shrink-0">
                {item.education_dates}
              </div>
            </div>
          </div>
        </a>
      </div>
    {/each}
  </div>
</section>
