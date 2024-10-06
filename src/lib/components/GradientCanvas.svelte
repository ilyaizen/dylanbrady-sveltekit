<script lang="ts">
  import { onMount } from 'svelte';
  import { darkMode } from '$lib/stores/darkMode';

  let canvas: HTMLCanvasElement;
  let context: CanvasRenderingContext2D;
  let time = 0;
  let width: number;
  let height: number;

  const color = (x: number, y: number, r: number, g: number, b: number) => {
    context.fillStyle = `rgb(${r}, ${g}, ${b})`;
    context.fillRect(x * 10, y * 10, 10, 10);
  };

  const R = (x: number, y: number, time: number) => {
    return Math.floor(192 + 64 * Math.cos((x * x - y * y) / 300 + time));
  };

  const G = (x: number, y: number, time: number) => {
    return Math.floor(192 + 64 * Math.sin((x * x * Math.cos(time / 4) + y * y * Math.sin(time / 3)) / 300));
  };

  const B = (x: number, y: number, time: number) => {
    return Math.floor(
      192 + 64 * Math.sin(5 * Math.sin(time / 9) + ((x - 100) * (x - 100) + (y - 100) * (y - 100)) / 1100)
    );
  };

  const startAnimation = () => {
    for (let x = 0; x <= 30; x++) {
      for (let y = 0; y <= 30; y++) {
        color(x, y, R(x, y, time), G(x, y, time), B(x, y, time));
      }
    }
    time += 0.03;
    requestAnimationFrame(startAnimation);
  };

  const handleResize = () => {
    width = window.innerWidth;
    height = window.innerHeight;
    canvas.width = width;
    canvas.height = height;
    context.scale(width / 310, height / 310);
  };

  $: overlayColor = $darkMode ? 'rgba(0, 0, 0, 0.5)' : 'rgba(255, 255, 255, 0.5)';

  onMount(() => {
    context = canvas.getContext('2d')!;
    handleResize();
    startAnimation();

    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize);
    };
  });
</script>

<canvas bind:this={canvas} />
<div class="overlay" style="background-color: {overlayColor};" />

<style>
  canvas {
    position: fixed;
    z-index: -2;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    filter: blur(50px);
  }

  .overlay {
    position: fixed;
    z-index: -1;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    transition: background-color 0.3s ease;
  }
</style>
