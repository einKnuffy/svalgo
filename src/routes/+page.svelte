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

		startPos[0] += offset[0];
		startPos[1] += offset[1];
	};

	// looking for any closer option
	/* const moveCell = () => {
		if (
			distance2D(startPos[0] + 1, startPos[1], objectivePos[0], objectivePos[1]) <
			distance2D(startPos[0], startPos[1], objectivePos[0], objectivePos[1])
		) {
			startPos[0] += 1;
		} else if (
			distance2D(startPos[0], startPos[1] + 1, objectivePos[0], objectivePos[1]) <
			distance2D(startPos[0], startPos[1], objectivePos[0], objectivePos[1])
		) {
			startPos[1] += 1;
		} else if (
			distance2D(startPos[0] - 1, startPos[1], objectivePos[0], objectivePos[1]) <
			distance2D(startPos[0], startPos[1], objectivePos[0], objectivePos[1])
		) {
			startPos[0] -= 1;
		} else if (
			distance2D(startPos[0], startPos[1] - 1, objectivePos[0], objectivePos[1]) <
			distance2D(startPos[0], startPos[1], objectivePos[0], objectivePos[1])
		) {
			startPos[1] -= 1;
		}
	}; */

	const startGame = async () => {
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

	const setPositions = (x, y) => {
		if (gameStarted) return;

		if (objectiveReached) {
			startPos = [-1, -1];
			objectivePos = [-1, -1];
			objectiveReached = false;
		}

		if (startPos[0] == -1 && startPos[1] == -1) startPos = [x, y];
		else if (objectivePos[0] == -1 && objectivePos[1] == -1) {
			objectivePos = [x, y];
			console.log('Start: ', startPos);
			console.log('Objective: ', objectivePos);

			startGame();
		}
	};
</script>

<div class="flex flex-col items-center my-8">
	<h1 class="text-3xl font-bold p-3">This is an experimental algorithmic visualizer</h1>
	<p>Made with ♥️ by <a href="https;//github.com/einKnuffy">einKnuffy</a></p>
</div>

<div class="flex items-center justify-center">
	<div class="flex bg-primary p-3 rounded-lg">
		{#each dataX as coordX}
			<div class="flex flex-col h-full">
				{#each dataY as coordY}
					<div
						class={`flex items-center justify-center md:w-8 md:h-8 sm:w-6 sm:h-6 bg-primary-focus ${
							objectivePos[0] == coordX && objectivePos[1] == coordY
								? objectiveReached
									? 'bg-green-500'
									: 'bg-red-500'
								: startPos[0] == coordX && startPos[1] == coordY && 'bg-white'
						} rounded-sm m-0.5`}
						aria-hidden
						on:click={() => setPositions(coordX, coordY)}
					/>
				{/each}
			</div>
		{/each}
	</div>
</div>

<style lang="postcss">
	:global(html) {
		background-color: theme(colors.gray.100);
	}
</style>
