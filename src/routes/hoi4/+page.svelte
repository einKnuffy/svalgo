<script lang="ts">
	import { onMount } from 'svelte';
	let canvas: HTMLCanvasElement;
	let ctx: CanvasRenderingContext2D | null = null;
	const width = 700;
	const height = 700;
	const pi_multiplier = Math.PI / 180;

	const numPolygons = 4;
	const maxVertices = 6;

	interface ScreenPoint {
		x: number;
		y: number;
	}

	interface Polygon {
		numVertices: number;
		objectVerticesX: number[];
		objectVerticesY: number[];
		objectVerticesZ: number[];
		screenVertices: ScreenPoint[];
		distanceToWorld: number;
		positionX: number;
		positionY: number;
		positionZ: number;
		color: number[];
	}

	const calculateDistance = (
		x1: number,
		y1: number,
		z1: number,
		x2: number,
		y2: number,
		z2: number
	) => {
		const deltaX: number = x2 - x1;
		const deltaY: number = y2 - y1;
		const deltaZ: number = z2 - z1;

		if (deltaX === 0 && deltaY === 0 && deltaZ === 0) {
			return 0;
		}

		return Math.sqrt(deltaX * deltaX + deltaY * deltaY + deltaZ * deltaZ);
	};

	const worldToScreen = (
		playerX: number,
		playerY: number,
		playerZ: number,
		viewAngleX: number,
		viewAngleY: number,
		viewAngleZ: number,
		polygon: Polygon
		// screenVertices: ScreenPoint
	) => {
		const cosX = Math.cos(viewAngleX);
		const sinX = Math.sin(viewAngleX);
		const cosY = Math.cos(viewAngleY);
		const sinY = Math.sin(viewAngleY);
		const cosZ = Math.cos(viewAngleZ);
		const sinZ = Math.sin(viewAngleZ);

		polygon.distanceToWorld = calculateDistance(
			playerX,
			playerY,
			playerZ,
			polygon.positionX,
			polygon.positionY,
			polygon.positionZ
		);
		const scale = (1 / (polygon.distanceToWorld + 1)) * 5;

		for (let i = 0; i < polygon.numVertices; i++) {
			let relativeX = polygon.objectVerticesX[i] - polygon.positionX;
			let relativeY = polygon.objectVerticesY[i] - polygon.positionY;
			let relativeZ = polygon.objectVerticesZ[i] - polygon.positionZ;

			let rotatedX = relativeX;
			let rotatedY = relativeY * cosX - relativeZ * sinX;
			let rotatedZ = relativeY * sinX + relativeZ * cosX;

			rotatedX = rotatedX * cosY + rotatedZ * sinY;
			rotatedZ = -rotatedX * sinY + rotatedZ * cosY;

			rotatedX = rotatedX * cosZ - rotatedY * sinZ;
			rotatedY = rotatedX * sinZ + rotatedY * cosZ;

			polygon.screenVertices[i].x = width / 2 + rotatedX * scale;
			polygon.screenVertices[i].y = height / 2 - rotatedY * scale;
		}
	};

	// A function to set a pixel to a random color
	function setPixel(x: number, y: number) {
		/* 	let r = Math.floor(Math.random() * 256);
		let g = Math.floor(Math.random() * 256);
		let b = Math.floor(Math.random() * 256);
		let a = Math.floor(Math.random() * 256); */
		const r = 255;
		const g = 0;
		const b = 0;
		const a = 255;

		let color = `rgba(${r}, ${g}, ${b}, ${a})`;

		if (!ctx) return;

		ctx.fillStyle = color;
		ctx.fillRect(x, y, 1, 1);
	}
	// A function to draw some pixels on the canvas
	function drawPixels() {
		/* 	for (let i = 0; i < 100; i++) {
			let x = Math.floor(Math.random() * width);
			let y = Math.floor(Math.random() * height);
			setPixel(x, y);
		} */

		for (let x = 0; x < width; x++) {
			for (let y = 0; y < height; y++) {
				setPixel(x, y);
			}
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
