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
		/* 	[1, 1],
		[1, -1],
		[-1, 1],
		[-1, -1] */
	];

	const moveCell = () => {
		let shortestDistance = Infinity;
		let offset = [0, 0];

		neighbours.forEach(([x, y]) => {
			let distance = distance2D(startPos[0] + x, startPos[1] + y, objectivePos[0], objectivePos[1]);
			if (distance < shortestDistance) {
				shortestDistance = distance;
				offset = [x, y];
			}
		});

		grid[startPos[0]][startPos[1]] = Cell.Empty;
		startPos[0] += offset[0];
		startPos[1] += offset[1];
		grid[startPos[0]][startPos[1]] = Cell.Start;
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
		{#if gameStarted}Game is running{:else if objectiveReached}Objective reached{:else}This is an
			experimental algorithmic visualizer{/if}
	</h1>
	<p>Made with ♥️ by <a href="https;//github.com/einKnuffy">einKnuffy</a></p>
</div>

<div class="flex items-center justify-center">
	<div class="flex bg-primary p-3 rounded-lg">
		{#each grid as row, coordX}
			<div class="flex flex-col h-full">
				{#each row as cell, coordY}
					<div
						class="flex items-center justify-center md:w-8 md:h-8 sm:w-6 sm:h-6 {cell ==
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
	<div class="flex flex-col ml-3">
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
