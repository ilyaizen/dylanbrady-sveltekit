<script lang="ts">
  import type { Content } from '@prismicio/client';

  export let slice: Content.ProjectsSlice;
</script>

<section
  class="max-w-xl mx-auto font-roboto pb-72"
  data-slice-type={slice.slice_type}
  data-slice-variation={slice.variation}
>
  <section id="projects">
    <div class="space-y-12 w-full py-12">
      <div style="opacity: 1; filter: blur(0px); transform: translateY(-6px) translateZ(0px);">
        <div class="flex flex-col items-center justify-center space-y-4 text-center">
          <div class="space-y-2">
            <div class="inline-block rounded-lg bg-foreground text-background px-3 py-1 text-sm">
              {slice.primary.projects_heading}
            </div>
            <h2 class="text-3xl font-bold tracking-tighter sm:text-5xl">{slice.primary.projects_title}</h2>
            <p class="text-muted-foreground md:text-xl/relaxed lg:text-base/relaxed xl:text-xl/relaxed">
              {slice.primary.projects_description}
            </p>
          </div>
        </div>
      </div>
    </div>
  </section>
</section>
