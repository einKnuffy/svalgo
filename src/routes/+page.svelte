<script>
	let dataX = Array(20)
		.fill(0)
		.map((_, i) => i);
	let dataY = Array(20)
		.fill(0)
		.map((_, i) => i);

	let gameStarted = false;
	let objectiveReached = false;
	let startPos = [-1, -1];
	let objectivePos = [-1, -1];

	const distance2D = (x1, y1, x2, y2) => Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));

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

	let allowedPaths = [];
	let prohibitedPaths = [];

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

		startPos[0] += offset[0];
		startPos[1] += offset[1];
	};

	const startGame = async () => {
		if (
			(startPos[0] == -1 && startPos[1] == -1) ||
			(objectivePos[0] == -1 && objectivePos[1] == -1)
		)
			return;

		console.log('Start: ', startPos);
		console.log('Objective: ', objectivePos);

		gameStarted = true;
		console.log(distance2D(startPos[0], startPos[1], objectivePos[0], objectivePos[1]));

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

	let selectAllowedPath = false;
	let selectProhibitedPath = false;

	const setPositions = (x, y) => {
		if (gameStarted) return;

		console.log(allowedPaths);
		console.log(prohibitedPaths);

		if (objectiveReached) {
			startPos = [-1, -1];
			objectivePos = [-1, -1];
			objectiveReached = false;
		}

		if (
			!allowedPaths.find((path) => path[0] == x && path[1] == y) &&
			!prohibitedPaths.find((path) => path[0] == x && path[1] == y)
		) {
			if (selectAllowedPath) {
				allowedPaths.push([x, y]);
			} else if (selectProhibitedPath) {
				prohibitedPaths.push([x, y]);
			}
		}

		if (selectAllowedPath || selectProhibitedPath) return;

		if (startPos[0] == -1 && startPos[1] == -1) startPos = [x, y];
		else if (objectivePos[0] == -1 && objectivePos[1] == -1) {
			objectivePos = [x, y];
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
		{#each dataX as coordX}
			<div class="flex flex-col h-full">
				{#each dataY as coordY}
					<div
						class={`flex items-center justify-center md:w-8 md:h-8 sm:w-6 sm:h-6 bg-primary-focus hover:bg-[#eeeeee20] ${
							/* 	objectivePos[0] == coordX && objectivePos[1] == coordY
								? objectiveReached
									? 'bg-green-500 hover:bg-green-500'
									: 'bg-red-500 hover:bg-red-500'
								: startPos[0] == coordX && startPos[1] == coordY
								? 'bg-white hover:bg-white'
								: null : allowedPaths.find((path) => path[0] == coordX && path[1] == coordY)
								? 'bg-red-500 hover:bg-red-500'
								: prohibitedPaths.find((path) => path[0] == coordX && path[1] == coordY)
								? 'bg-green-500 hover:bg-green-500'
								: '#eee' */
							objectivePos[0] == coordX && objectivePos[1] == coordY && objectiveReached
								? 'bg-green-500'
								: objectivePos[0] == coordX && objectivePos[1] == coordY && !objectiveReached
								? 'bg-red-500'
								: startPos[0] == coordX && startPos[1] == coordY
								? 'bg-secondary-content'
								: /* 	: allowedPaths.includes(() => path[0] == coordX && path[1] == coordY) != null
								? 'bg-red-500 hover:bg-red-500' */
								prohibitedPaths.find((path) => path[0] == coordX && path[1] == coordY) != null
								? 'bg-green-500 hover:bg-green-500'
								: null
						} rounded-sm m-0.5`}
						aria-hidden
						on:click={() => setPositions(coordX, coordY)}
					/>
				{/each}
			</div>
		{/each}
	</div>
	<div class="flex flex-col ml-3">
		<button
			class="btn btn-primary text-opacity-50"
			on:click={() => (selectAllowedPath = !selectAllowedPath)}>Path</button
		>
		<button
			class="btn btn-primary text-opacity-50 mt-2"
			disabled={selectProhibitedPath}
			on:click={() => (selectProhibitedPath = !selectProhibitedPath)}>Prohibited</button
		>
		<button
			class="btn btn-success text-opacity-50 mt-12"
			disabled={(startPos[0] == -1 && startPos[1] == -1) ||
				(objectivePos[0] == -1 && objectivePos[1] == -1)}
			on:click={startGame}>Start</button
		>
	</div>
</div>

<style lang="postcss">
	:global(html) {
		background-color: theme(colors.gray.100);
	}
</style>
