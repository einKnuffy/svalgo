<script lang="ts">
	import { onMount } from 'svelte';

	// Constants
	const WINDOW_WIDTH = 800;
	const WINDOW_HEIGHT = 800;

	// Angle conversion constant
	const PI_MULTIPLIER = Math.PI / 180.0;

	// POLYGONS & numVertices
	const NUM_POLYGONS = 4;
	const MAX_VERTICES = 6;

	// Structure for 2D screen coordinates
	interface ScreenPoint {
		x: number;
		y: number;
	}

	// Structure for polygons
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

	const EXPERIMENTAL_MULT = 3;

	function calculateDistance(
		x1: number,
		y1: number,
		z1: number,
		x2: number,
		y2: number,
		z2: number
	): number {
		const deltaX = x2 - x1;
		const deltaY = y2 - y1;
		const deltaZ = z2 - z1;

		if (deltaX === 0 && deltaY === 0 && deltaZ === 0) return 0;

		return Math.sqrt(deltaX * deltaX + deltaY * deltaY + deltaZ * deltaZ);
	}

	function worldToScreen(
		playerX: number,
		playerY: number,
		playerZ: number,
		viewAngleX: number,
		viewAngleY: number,
		viewAngleZ: number,
		polygon: Polygon,
		screenVertices: ScreenPoint[]
	): void {
		const cosX = Math.cos(viewAngleX);
		const sinX = Math.sin(viewAngleX);
		const cosY = Math.cos(viewAngleY);
		const sinY = Math.sin(viewAngleY);
		const cosZ = Math.cos(viewAngleZ);
		const sinZ = Math.sin(viewAngleZ);

		polygon.distanceToWorld = calculateDistance(0, 0, polygon.positionZ, 0, 0, playerZ);
		const scale = 1.0 / ((1.0 + Math.abs(polygon.distanceToWorld)) * 15.0);

		for (let i = 0; i < polygon.numVertices; i++) {
			const relativeX = polygon.objectVerticesX[i] - polygon.positionX;
			const relativeY = polygon.objectVerticesY[i] - polygon.positionY;
			const relativeZ = polygon.objectVerticesZ[i] - polygon.positionZ;

			let rotatedX = relativeX;
			let rotatedY = relativeY * cosX - relativeZ * sinX;
			let rotatedZ = relativeY * sinX + relativeZ * cosX;

			let tempX = rotatedX * cosY + rotatedZ * sinY;
			let tempZ = -rotatedX * sinY + rotatedZ * cosY;
			rotatedX = tempX;
			rotatedZ = tempZ;

			let tempX2 = rotatedX * cosZ - rotatedY * sinZ;
			let tempY = rotatedX * sinZ + rotatedY * cosZ;
			rotatedX = tempX2;
			rotatedY = tempY;

			const screenX = WINDOW_WIDTH / 2.0 + rotatedX * scale;
			const screenY = WINDOW_HEIGHT / 2.0 + rotatedY * scale;

			screenVertices[i] = { x: screenX, y: screenY };
		}
	}

	function isInsidePolygon(
		screenVertices: ScreenPoint[],
		numVertices: number,
		x: number,
		y: number
	): boolean {
		let c = false;
		for (let i = 0, j = numVertices - 1; i < numVertices; j = i++) {
			if (
				screenVertices[i].y > y !== screenVertices[j].y > y &&
				x <
					((screenVertices[j].x - screenVertices[i].x) * (y - screenVertices[i].y)) /
						(screenVertices[j].y - screenVertices[i].y) +
						screenVertices[i].x
			) {
				c = !c;
			}
		}
		return c;
	}

	function fillPolygon(renderer: CanvasRenderingContext2D, polygon: Polygon): void {
		for (let y = 0; y < WINDOW_HEIGHT; y++) {
			for (let x = 0; x < WINDOW_WIDTH; x++) {
				if (isInsidePolygon(polygon.screenVertices, polygon.numVertices, x, y)) {
					renderer.fillStyle = `rgba(${polygon.color[0]}, ${polygon.color[1]}, ${polygon.color[2]}, ${polygon.color[3]})`;
					renderer.fillRect(x, y, 1, 1);
				}
			}
		}
	}

	function createSamplePolygons(): Polygon[] {
		const polygons: Polygon[] = [];

		const objectVerticesX = [
			[-6.0, 4.0, 4.0, -6.0],
			[4.0, 4.0, 4.0, 4.0],
			[4.0, 4.0, -6.0, -6.0],
			[-6.0, 4.0, -6.0, -6.0]
			// Add more polygons here
		];
		const objectVerticesY = [
			[0.0, 0.0, 7.0, 7.0],
			[0.0, 0.0, 7.0, 7.0],
			[0.0, 0.0, 7.0, 7.0],
			[0.0, 0.0, 7.0, 7.0]
			// Add more polygons here
		];
		const objectVerticesZ = [
			[0.0, 0.0, 0.0, 0.0],
			[0.0, 0.0, -6.0, -6.0],
			[-6.0, -6.0, -6.0, -6.0],
			[-6.0, -6.0, 0.0, 0.0]
			// Add more polygons here
		];

		for (let i = 0; i < NUM_POLYGONS; i++) {
			const numVertices = objectVerticesX[i].length;

			const polygon: Polygon = {
				numVertices,
				objectVerticesX: objectVerticesX[i],
				objectVerticesY: objectVerticesY[i],
				objectVerticesZ: objectVerticesZ[i],
				screenVertices: [],
				distanceToWorld: 0.0,
				positionX: 0.0,
				positionY: 0.0,
				positionZ: 0.0,
				color: [0, 0, 0, 0]
			};

			if (i === 0) {
				polygon.color = [255, 0, 0, 255];
			} else if (i === 1) {
				polygon.color = [0, 255, 0, 255];
			} else {
				polygon.color = [0, 0, 255, 255];
			}

			polygons.push(polygon);
		}

		return polygons;
	}

	function partition(polygons: Polygon[], low: number, high: number): number {
		const pivot = polygons[high].distanceToWorld;
		let i = low - 1;

		for (let j = low; j < high; j++) {
			if (polygons[j].distanceToWorld > pivot) {
				i++;
				const temp = polygons[i];
				polygons[i] = polygons[j];
				polygons[j] = temp;
			}
		}
		const temp = polygons[i + 1];
		polygons[i + 1] = polygons[high];
		polygons[high] = temp;
		return i + 1;
	}

	function quickSort(polygons: Polygon[], low: number, high: number): void {
		if (low < high) {
			const pivot = partition(polygons, low, high);
			quickSort(polygons, low, pivot - 1);
			quickSort(polygons, pivot + 1, high);
		}
	}

	function display(renderer: CanvasRenderingContext2D, polygons: Polygon[]): void {
		// Clear the screen
		renderer.fillStyle = '#000000';
		renderer.fillRect(0, 0, WINDOW_WIDTH, WINDOW_HEIGHT);

		for (let i = 0; i < NUM_POLYGONS; i++) {
			fillPolygon(renderer, polygons[i]);
		}
	}

	let canvas: HTMLCanvasElement;
	let renderer: CanvasRenderingContext2D;

	onMount(() => {
		canvas = document.getElementById('canvas') as HTMLCanvasElement;
		if (!canvas) return;

		renderer = canvas.getContext('2d') as CanvasRenderingContext2D;
		if (!renderer) return;

		canvas.width = WINDOW_WIDTH;
		canvas.height = WINDOW_HEIGHT;

		if (!renderer) return;

		const polygons = createSamplePolygons();

		let viewAngleX = 0.0;
		let viewAngleY = 0.0;
		let viewAngleZ = 0.0;

		let worldX = 0.0;
		let worldY = 0.0;
		let worldZ = 0.0;

		// Enter the main loop
		let quit = false;
		function mainLoop() {
			viewAngleY += 1.0;
			viewAngleX += 0.1;

			for (let i = 0; i < NUM_POLYGONS; i++) {
				worldToScreen(
					worldX,
					worldY,
					worldZ,
					viewAngleX * PI_MULTIPLIER,
					viewAngleY * PI_MULTIPLIER,
					viewAngleZ * PI_MULTIPLIER,
					polygons[i],
					polygons[i].screenVertices
				);
			}

			quickSort(polygons, 0, NUM_POLYGONS - 1);
			display(renderer, polygons);

			// setTimeout(mainLoop, 100);
		}

		// mainLoop();
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
