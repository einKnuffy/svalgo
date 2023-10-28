<script lang="ts">
	import { onMount } from 'svelte';
	let canvas: HTMLCanvasElement;
	let ctx: CanvasRenderingContext2D | null = null;
	let width = 700;
	let height = 700;

	// A function to set a pixel to a random color
	function setPixel(x: number, y: number) {
		let r = Math.floor(Math.random() * 256);
		let g = Math.floor(Math.random() * 256);
		let b = Math.floor(Math.random() * 256);
		let a = Math.floor(Math.random() * 256);
		let color = `rgba(${r}, ${g}, ${b}, ${a})`;

		if (!ctx) return;

		ctx.fillStyle = color;
		ctx.fillRect(x, y, 1, 1);
	}
	// A function to draw some pixels on the canvas
	function drawPixels() {
		for (let i = 0; i < 100; i++) {
			let x = Math.floor(Math.random() * width);
			let y = Math.floor(Math.random() * height);
			setPixel(x, y);
		}
	}

	// A function to get the canvas context and draw the pixels
	function initCanvas() {
		canvas = document.getElementById('canvas') as HTMLCanvasElement;
		ctx = canvas.getContext('2d');
		drawPixels();
	}

	// A hook to run the initCanvas function after the component is mounted
	onMount(() => {
		initCanvas();
	});
</script>

<div class="text-center">
	<h1 class="text-4xl font-bold">Svalgo</h1>
	<h2 class="text-xl text-gray-500">Hearts of Iron IV</h2>
</div>

<canvas width="700" height="700" class="mx-auto" id="canvas" />

<style lang="postcss">
	:global(html) {
		background-color: theme(colors.gray.100);
	}

	canvas {
		border: 1px solid theme(colors.gray.300);
		border-radius: 1rem;
	}
</style>
