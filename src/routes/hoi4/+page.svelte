<script lang="ts">
	import { onMount } from 'svelte';
	let canvas: HTMLCanvasElement;
	let ctx: CanvasRenderingContext2D | null = null;

	const WINDOW_WIDTH = 700;
	const WINDOW_HEIGHT = 700;

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

	const numVertices = [4, 4, 4, 4];
	const objectVerticesX: number[][] = [];
	const objectVerticesY: number[][] = [];
	const objectVerticesZ: number[][] = [];

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

		if (deltaX === 0 && deltaY === 0 && deltaZ === 0) {
			return 0;
		}

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

			const screenX = Math.floor(WINDOW_WIDTH / 2.0 + rotatedX * scale);
			const screenY = Math.floor(WINDOW_HEIGHT / 2.0 + rotatedY * scale);

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

	function fillPolygon(context: CanvasRenderingContext2D, polygon: Polygon): void {
		for (let y = 0; y < WINDOW_HEIGHT; y++) {
			for (let x = 0; x < WINDOW_WIDTH; x++) {
				if (isInsidePolygon(polygon.screenVertices, polygon.numVertices, x, y)) {
					context.fillStyle = `rgba(${polygon.color[0]}, ${polygon.color[1]}, ${polygon.color[2]}, ${polygon.color[3]})`;
					context.fillRect(x, y, 1, 1);
				}
			}
		}
	}

	function createSamplePolygons(): Polygon[] {
		objectVerticesX[0] = [-6.0, 4.0, 4.0, -6.0];
		objectVerticesY[0] = [0.0, 0.0, 7.0, 7.0];
		objectVerticesZ[0] = [0.0, 0.0, 0.0, 0.0];

		objectVerticesX[1] = [4.0, 4.0, 4.0, 4.0];
		objectVerticesY[1] = [0.0, 0.0, 7.0, 7.0];
		objectVerticesZ[1] = [0.0, -6.0, -6.0, 0.0];

		const polygons: Polygon[] = [];

		for (let i = 0; i < numVertices.length; i++) {
			const numVerticesPerPolygon = numVertices[i];

			let polygon: Polygon = {
				numVertices: numVerticesPerPolygon,
				objectVerticesX: objectVerticesX[i],
				objectVerticesY: objectVerticesY[i],
				objectVerticesZ: objectVerticesZ[i],
				screenVertices: ScreenPoint[],
				distanceToWorld: 0.0,
				positionX: 0.0,
				positionY: 0.0,
				positionZ: 0.0,
				color: []
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

	function display(context: CanvasRenderingContext2D, polygons: Polygon[]): void {
		context.fillStyle = '#000000';
		context.fillRect(0, 0, WINDOW_WIDTH, WINDOW_HEIGHT);

		for (const polygon of polygons) {
			fillPolygon(context, polygon);
		}
	}

	// A function to get the canvas context and draw the pixels
	function initCanvas() {
		canvas = document.getElementById('canvas') as HTMLCanvasElement;
		canvas.width = WINDOW_WIDTH;
		canvas.height = WINDOW_HEIGHT;
		const context = canvas.getContext('2d');

		if (context) {
			const polygons = createSamplePolygons();

			while (true) {
				display(context, polygons);
			}
		}
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
