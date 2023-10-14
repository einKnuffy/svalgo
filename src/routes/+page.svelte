<!-- <script lang="ts">
	enum Cell {
		Empty = 0,
		Start = 1,
		Objective = 2,
		Obstacle = 3
	}

	const gridSize = 25;
	let grid = Array(gridSize)
		.fill(Cell.Empty)
		.map(() => Array(gridSize).fill(Cell.Empty));

	let gameStarted = false;
	let startPos: number[] = [-1, -1];
	let objectivePos: number[] = [-1, -1];
	let path: any[] = []; // Stores the path found by A* algorithm

	// Define a heuristic function for A* (Euclidean distance)
	const heuristic = (x: number, y: number) => {
		const dx = x - objectivePos[0];
		const dy = y - objectivePos[1];
		return Math.sqrt(dx * dx + dy * dy);
	};

	// Define a priority queue for open nodes in A* (sorted by f-score)
	const openSet: [number, number, number][] = []; // [x, y, fScore]
	openSet.push([...startPos, heuristic(startPos[0], startPos[1])]);

	const cameFrom: number[][] = Array(gridSize)
		.fill([])
		.map(() => Array(gridSize).fill(-1)); // Store the previous node in the best path

	const endGame = () => {
		gameStarted = false;
		path = [];
	};

	const neighbors = [
		[1, 0],
		[-1, 0],
		[0, 1],
		[0, -1]
	];

	const aStar = () => {
		while (openSet.length > 0) {
			// Find the node with the lowest fScore in openSet
			openSet.sort((a, b) => a[2] - b[2]);
			const [currentX, currentY, _] = openSet.shift();

			if (currentX === objectivePos[0] && currentY === objectivePos[1]) {
				// Reconstruct the path and set it
				path = [];
				let x = currentX;
				let y = currentY;
				while (x !== startPos[0] || y !== startPos[1]) {
					path.push([x, y]);
					const [prevX, prevY] = cameFrom[x][y];
					x = prevX;
					y = prevY;
				}
				path.reverse();
				return;
			}

			for (const [dx, dy] of neighbors) {
				const neighborX = currentX + dx;
				const neighborY = currentY + dy;

				if (
					neighborX >= 0 &&
					neighborX < gridSize &&
					neighborY >= 0 &&
					neighborY < gridSize &&
					grid[neighborX][neighborY] !== Cell.Obstacle
				) {
					const tentativeGScore = cameFrom[currentX][currentY] + 1;
					if (
						tentativeGScore < cameFrom[neighborX][neighborY] ||
						cameFrom[neighborX][neighborY] === -1
					) {
						cameFrom[neighborX][neighborY] = [currentX, currentY];
						const fScore = tentativeGScore + heuristic(neighborX, neighborY);
						openSet.push([neighborX, neighborY, fScore]);
					}
				}
			}
		}
	};

	const moveCell = async () => {
		if (path.length === 0) {
			aStar();
		}

		if (path.length > 0) {
			const [x, y] = path.shift();
			grid[startPos[0]][startPos[1]] = Cell.Empty;
			startPos = [x, y];
			grid[startPos[0]][startPos[1]] = Cell.Start;
		}

		await new Promise((resolve) => setTimeout(resolve, 200));
	};

	const startGame = async () => {
		if (
			startPos[0] === -1 ||
			startPos[1] === -1 ||
			objectivePos[0] === -1 ||
			objectivePos[1] === -1
		) {
			return;
		}

		gameStarted = true;
		aStar();
		moveCell();
		endGame();
	};

	let selectObstacle = false;

	const setPositions = (x: number, y: number) => {
		if (gameStarted) {
			return;
		}

		if (grid[x][y] === Cell.Empty) {
			if (selectObstacle) {
				grid[x][y] = Cell.Obstacle;
			} else if (startPos[0] === -1 && startPos[1] === -1) {
				startPos = [x, y];
				grid[x][y] = Cell.Start;
			} else if (objectivePos[0] === -1 && objectivePos[1] === -1) {
				objectivePos = [x, y];
				grid[x][y] = Cell.Objective;
			}
		}
	};
</script> -->

<script lang="ts">
	enum Cell {
		Empty = 0,
		Start = 1,
		Objective = 2,
		Obstacle = 3
	}

	const gridSize = 25;
	let grid = Array(gridSize)
		.fill(Cell.Empty)
		.map(() => Array(gridSize).fill(Cell.Empty));

	let gameStarted = false;
	let objectiveReached = false;
	let startPos = [-1, -1];
	let objectivePos = [-1, -1];

	const distance2D = (x1: number, y1: number, x2: number, y2: number) =>
		Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));

	const endGame = () => {
		gameStarted = false;
		// startPos = [-1, -1];
		// objectivePos = [-1, -1];
	};

	const neighbours = [
		[1, 0],
		[-1, 0],
		[0, 1],
		[0, -1]
	];

	let walkedPaths;

	const moveCell = (posX = startPos[0], posY = startPos[1], score = 0) => {
		let shortestDistance = Infinity;
		let offset = [0, 0];
		let neighbourIsObstacle = false;

		neighbours.forEach(([x, y]) => {
			if (grid[posX + x][posY + y] == Cell.Obstacle) {
				neighbourIsObstacle = true;
				return;
			}

			let distance = distance2D(posX + x, posY + y, objectivePos[0], objectivePos[1]);

			if (distance < shortestDistance) {
				shortestDistance = distance;
				offset = [x, y];
				score++;
			}
		});

		// do it
		// spawn all new cells and if no neibours with obstacles around, delete the two most distant
		// then spawn on these one news, etc. global
		// if you have a straight line of obstacles that shouldnt be crossed, then walk that line "walk by reducing distant to the obstacle while walking all ways around (vertical and x), if then you get around and you are faster than the others"
		// then delete the others again and continue walking
		// BASIC ALGO

		if (score < 0) return;

		grid[startPos[0]][startPos[1]] = Cell.Empty;
		startPos[0] += offset[0];
		startPos[1] += offset[1];
		grid[startPos[0]][startPos[1]] = Cell.Start;

		// grid[posX][posY] = Cell.Start;
		// await new Promise((resolve) => setTimeout(resolve, 200));
		/* 		moveCell(posX + offset[0], posY + offset[1], score); */
	};

	const startGame = async () => {
		if (
			(startPos[0] == -1 && startPos[1] == -1) ||
			(objectivePos[0] == -1 && objectivePos[1] == -1)
		)
			return;

		console.log('Grid: ', grid);

		console.log('Start: ', startPos);
		console.log('Objective: ', objectivePos);
		gameStarted = true;

		while (!objectiveReached) {
			if (distance2D(startPos[0], startPos[1], objectivePos[0], objectivePos[1]) == 0) {
				objectiveReached = true;
				break;
			}

			await new Promise((resolve) => setTimeout(resolve, 500));
			moveCell();
		}

		endGame();
	};

	let selectObstacle = false;
	const setPositions = (x: number, y: number) => {
		if (gameStarted) return;

		if (objectiveReached) {
			startPos = [-1, -1];
			objectivePos = [-1, -1];
			objectiveReached = false;
			grid = Array(gridSize)
				.fill(Cell.Empty)
				.map(() => Array(gridSize).fill(Cell.Empty));
		}

		if (grid[x][y] == Cell.Empty) {
			if (selectObstacle) {
				grid[x][y] = Cell.Obstacle;
			} else if (startPos[0] == -1 && startPos[1] == -1) {
				startPos = [x, y];
				grid[x][y] = Cell.Start;
			} else if (objectivePos[0] == -1 && objectivePos[1] == -1) {
				objectivePos = [x, y];
				grid[x][y] = Cell.Objective;
			}
		}
	};
</script>

<div class="flex flex-col items-center my-8">
	<h1 class="text-3xl font-bold p-3">
		{#if gameStarted}Game is running{:else if objectiveReached}Objective reached{:else}Svalgo{/if}
	</h1>
	<p>Made with ♥️ by <a href="https;//github.com/einKnuffy">einKnuffy</a></p>
</div>

<div class="flex flex-col space-y-3 md:space-x-3 md:flex-row items-center justify-center">
	<div class="flex bg-primary p-3 aspect-square rounded-lg">
		{#each grid as row, coordX}
			<div class="flex flex-col h-full w-full">
				{#each row as cell, coordY}
					<div
						class="flex items-center justify-center lg:w-6 lg:h-6 md:w-5 md:h-5 w-3 h-3 {cell ==
						Cell.Objective
							? objectiveReached
								? 'bg-green-500 hover:bg-green-500'
								: 'bg-red-500 hover:bg-red-500'
							: cell == Cell.Start
							? 'bg-white hover:bg-white'
							: cell == Cell.Obstacle
							? 'bg-blue-900'
							: 'bg-primary-focus hover:bg-white hover:bg-opacity-20'} rounded-sm m-0.5"
						id="{coordX}-{coordY}"
						aria-hidden
						on:click={() => setPositions(coordX, coordY)}
					/>
				{/each}
			</div>
		{/each}
	</div>
	<div class="flex flex-col">
		<button
			class="btn btn-primary text-opacity-50 mt-2"
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
