<script lang="ts">
  import type { Content } from '@prismicio/client';
  import { onMount } from 'svelte';
  import gsap from 'gsap';

  export let slice: Content.WorkSlice;

  let tl: gsap.core.Timeline;

  onMount(() => {
    tl = gsap.timeline({ paused: true });

    tl.fromTo('.work-heading-animation', { x: -5, opacity: 0 }, { x: 0, opacity: 1, duration: 0.4, ease: 'expo.out' })
      .fromTo(
        '.work-item-animation',
        { y: 20, opacity: 0 },
        { y: 0, opacity: 1, duration: 0.6, stagger: 0.05, ease: 'expo.out' },
        '-=0.4'
      )
      .call(() => {
        // Dispatch a custom event when work animation is complete
        window.dispatchEvent(new CustomEvent('workAnimationComplete'));
      });

    // Start animation when hero animation completes
    window.addEventListener('heroAnimationComplete', startWorkAnimation);

    return () => {
      // Clean up the event listener when the component is destroyed
      window.removeEventListener('heroAnimationComplete', startWorkAnimation);
    };
  });

  function startWorkAnimation() {
    tl.play();
  }

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
  class="max-w-xl mx-auto font-roboto pt-8"
  id="work"
  data-slice-type={slice.slice_type}
  data-slice-variation={slice.variation}
>
  <h2 class="work-heading-animation text-lg sm:text-xl font-bold pb-4">{slice.primary.work_heading}</h2>
  <div>
    {#each slice.primary.work_item as item}
      <div class="work-item-animation">
        <a class="block cursor-pointer pb-4" href={item.work_url}>
          <div class="rounded-full bg-card/20 text-card-foreground flex items-center">
            <div class="flex-none">
              <span class="relative flex shrink-0 overflow-hidden size-12 m-auto">
                <div
                  class="w-full h-full"
                  style={getIconStyle(item.work_icon_color ?? '', item.work_icon?.url ?? '')}
                ></div>
              </span>
            </div>
            <div class="flex-grow ml-4 flex items-center justify-between">
              <div class="flex flex-col">
                <h3 class="inline-flex items-center justify-start font-semibold leading-none text-xs sm:text-sm group">
                  {item.work_title}
                </h3>
                <div class="font-sans text-xs">{item.work_description}</div>
              </div>
              <div class="text-xs sm:text-sm tabular-nums text-muted-foreground text-right ml-4 mr-4 shrink-0">
                {item.work_dates}
              </div>
            </div>
          </div>
        </a>
      </div>
    {/each}
  </div>
</section>
