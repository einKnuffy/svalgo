<script lang="ts">
	enum Cell {
		Empty = 0,
		Start = 1,
		Objective = 2,
		Obstacle = 3,
		Path = 4 // Added to track the path
	}

	class Node {
		position: number[];
		gCost: number;
		hCost: number;
		fCost: number;
		parent: Node | null;

		constructor(position: number[], gCost: number, hCost: number) {
			this.position = position;
			this.gCost = gCost;
			this.hCost = hCost;
			this.fCost = gCost + hCost;
			this.parent = null;
		}
	}

	const gridSize = 25;
	let grid = Array(gridSize)
		.fill(Cell.Empty)
		.map(() => Array(gridSize).fill(Cell.Empty));

	let gameStarted = false;
	let startPos: number[] = [-1, -1];
	let objectivePos: number[] = [-1, -1];

	const getNeighbours = (node: Node) => {
		const movements = [
			{ dx: 1, dy: 0 },
			{ dx: -1, dy: 0 },
			{ dx: 0, dy: 1 },
			{ dx: 0, dy: -1 }
		];

		const neighbors = [];
		for (const movement of movements) {
			const neighborPosition = [node.position[0] + movement.dx, node.position[1] + movement.dy];

			if (isValidPosition(neighborPosition)) {
				neighbors.push(new Node(neighborPosition, 0, calculateHeuristic(neighborPosition)));
			}
		}
		return neighbors;
	};

	const calculateHeuristic = (pos: number[]) => {
		const dx = pos[0] - objectivePos[0];
		const dy = pos[1] - objectivePos[1];
		return Math.sqrt(dx * dx + dy * dy);
	};

	const isValidPosition = (pos: number[]) => {
		return (
			pos[0] >= 0 &&
			pos[0] < gridSize &&
			pos[1] >= 0 &&
			pos[1] < gridSize &&
			grid[pos[0]][pos[1]] !== Cell.Obstacle
		);
	};

	const reconstructPath = (node: Node) => {
		const path = [];
		while (node.parent) {
			path.unshift(node.position);
			node = node.parent;
		}

		return path;
	};

	const startGame = () => {
		if (
			startPos[0] === -1 ||
			startPos[1] === -1 ||
			objectivePos[0] === -1 ||
			objectivePos[1] === -1
		) {
			return;
		}

		gameStarted = true;

		const openSet: Node[] = [new Node(startPos, 0, calculateHeuristic(startPos))];
		const closedSet = new Set();

		while (openSet.length > 0) {
			openSet.sort((a, b) => a.fCost - b.fCost);
			const currentNode = openSet.shift();
			if (!currentNode) break;

			if (
				currentNode.position[0] === objectivePos[0] &&
				currentNode.position[1] === objectivePos[1]
			) {
				// Reconstruct the path and set it
				const path = reconstructPath(currentNode);
				for (const [x, y] of path) {
					grid[x][y] = Cell.Path;
				}
				break;
			}

			const neighbors = getNeighbours(currentNode);
			if (neighbors.length === 0) break;

			for (const neighbor of neighbors) {
				if (closedSet.has(neighbor.position)) {
					continue;
				}

				const tentativeGCost = currentNode.gCost + 1;
				if (
					!openSet.some(
						(node) =>
							node.position[0] === neighbor.position[0] && node.position[1] === neighbor.position[1]
					) ||
					tentativeGCost < neighbor.gCost
				) {
					neighbor.parent = currentNode;
					neighbor.gCost = tentativeGCost;
					neighbor.fCost = neighbor.gCost + neighbor.hCost;

					if (
						!openSet.some(
							(node) =>
								node.position[0] === neighbor.position[0] &&
								node.position[1] === neighbor.position[1]
						)
					) {
						openSet.push(neighbor);
					}
				}
			}
		}

		endGame();
	};

	const endGame = () => {
		gameStarted = false;
	};

	let selectObstacle = false;
	const setPositions = (x: number, y: number) => {
		if (gameStarted) {
			return;
		}

		if (grid[x][y] === Cell.Empty) {
			if (selectObstacle) {
				grid[x][y] = Cell.Obstacle;
			}
			if (startPos[0] === -1 && startPos[1] === -1) {
				startPos = [x, y];
				grid[x][y] = Cell.Start;
			} else if (objectivePos[0] === -1 && objectivePos[1] === -1) {
				objectivePos = [x, y];
				grid[x][y] = Cell.Objective;
			}
		}
	};
</script>

<div class="text-center">
	<h1 class="text-4xl font-bold">Svalgo</h1>
	<h2 class="text-xl text-gray-500">
		{#if gameStarted}Game is running: {:else}A* Algorithm{/if}
	</h2>
</div>

<div class="mt-8 flex flex-col space-y-3 md:space-x-3 md:flex-row items-center justify-center">
	<div class="flex bg-primary p-3 aspect-square rounded-lg">
		{#each grid as row, coordX}
			<div class="flex flex-col h-full w-full">
				{#each row as cell, coordY}
					<div
						class="flex items-center justify-center lg:w-6 lg:h-6 md:w-5 md:h-5 w-3 h-3 {cell ==
						Cell.Objective
							? 'bg-red-500 hover:bg-red-500'
							: cell == Cell.Start
							? 'bg-pink-200 hover:bg-pink-200'
							: cell == Cell.Obstacle
							? 'bg-blue-900'
							: cell == Cell.Path
							? 'bg-white hover:bg-white'
							: 'bg-primary-focus hover:bg-white hover:bg-opacity-20'} rounded-sm m-0.5"
						on:click={() => setPositions(coordX, coordY)}
					/>
				{/each}
			</div>
		{/each}
	</div>
	<div class="flex flex-col">
		<button
			class="btn btn-primary text-opacity-50 mt-2 {selectObstacle ? 'btn-active' : 'btn-ghost'}"
			on:click={() => (selectObstacle = !selectObstacle)}>Obstacle</button
		>
		<button
			class="btn btn-success text-opacity-50 mt-12"
			disabled={gameStarted}
			on:click={startGame}>Start</button
		>
	</div>
</div>

<style lang="postcss">
	:global(html) {
		background-color: theme(colors.gray.100);
	}
</style>
