<script>
	import { onMount, tick } from 'svelte';
	import { fade, draw, fly } from 'svelte/transition';
	import { spring } from 'svelte/motion';
	import axios from 'axios';
	import Modal from './Modal.svelte';

	axios.defaults.allowCredentials = true;

	let linesVisible = false;

	onMount(() => {
		setTimeout(() => {
			linesVisible = true;
		}, 1000);
	});

	let radius = 3;
	let radiusSpring = spring(3);
	$: radiusSpring.set(linesVisible ? 3 : 0);

	let lineWidth = 2;
	
	let dotsHorizontalCount = 4;
	let dotsVerticalCount = 4;
	let player = 1;

	let difficulty = 0; // 0 = easy, 1 = medium, 2 = hard
	
	$: spaceBetween = 100 / Math.max(dotsHorizontalCount, dotsVerticalCount);
	$: offset = (100 - (Math.max(dotsHorizontalCount, dotsVerticalCount) - 1) * spaceBetween) / 2;
	
	$: horizontalLines = Array(dotsVerticalCount).fill()
		.map(() => Array(dotsHorizontalCount - 1).fill().map(() => 0));

	$: verticalLines = Array(dotsHorizontalCount).fill()
		.map(() => Array(dotsVerticalCount - 1).fill().map(() => 0));
	
	$: dotsHorizontal = Array(dotsHorizontalCount).fill().map((_, i) => i);
	$: dotsVertical = Array(dotsVerticalCount).fill().map((_, i) => i);

	$: boxes = Array(dotsVerticalCount - 1).fill().map(() => Array(dotsHorizontalCount - 1).fill(0));
	// $: console.log(boxes);

	$: win = ![...horizontalLines.flatMap(x => x), ...verticalLines.flatMap(x => x)].some(x => x === 0);

	let showModal = false;
	
	async function fillBoxes() {
		let filled = false;
		for (let i = 0; i < dotsHorizontalCount - 1; i++) {
            for (let j = 0; j < dotsVerticalCount - 1; j++) {
                if(boxes[i][j] === 0) {
                    if((horizontalLines[i][j] != 0) && (horizontalLines[i + 1][j] != 0) && (verticalLines[j][i] != 0) && (verticalLines[j + 1][i] != 0)) {
						await tick();
						boxes[i][j] = player;
						filled = true;
                    }
                }
            }
		}
		
		if (!filled) {
			player *= -1;
		}
		checkWin();
	}

	async function checkWin() {
		await tick();
		showModal = win;
	}

	async function sendToServer() {
		if (player === 1) return;
		const data = {
			difficulty,
			horizontalLines,
			verticalLines,
		};
		try {
			const response = await axios.post('/ai/', data);
			console.log('RESPONSE', response.data);
			if (!response.data.error) {
				const { side, i, j } = response.data;
				if (side === 0) {
					// horizontalLines[i][j] = -1;
					handleHorizontalLineClick(i, j);
				} else {
					// verticalLines[i][j] = -1;
					handleVerticalLineClick(i, j);
				}
			}
		} catch (e) {
			console.log('ERRORRRRRRRR');
			console.log(Object.keys(e), e.response);
		}
	}

	function handleHorizontalLineClick(row, col) {
		// console.log('horizontal', row, col)
		if (horizontalLines[row][col] !== 0) return;
		horizontalLines[row][col] = player;
		fillBoxes();
		sendToServer();
	}
	
	function handleVerticalLineClick(row, col) {
		// console.log('vertical', row, col)
		if (verticalLines[row][col] !== 0) return;
		verticalLines[row][col] = player;
		fillBoxes();
		sendToServer();
	}

	function resetGame() {
		linesVisible = false;
		setTimeout(() => {
			horizontalLines = horizontalLines.map(lines => lines.map(() => 0));
			verticalLines = verticalLines.map(lines => lines.map(() => 0));
			boxes = boxes.map(list => list.map(() => 0));
			player = 1;
			linesVisible = true;
		}, 500);
	}
</script>

<h1 style="color: {player === 1 ? '#0050b3' : '#f5222d'};">
	{player === 1 ? 'Player 1' : 'Player 2'}
</h1>
<div class="score">
	<span class="score1">0</span> | <span class="score2">0</span>
</div>

<div class="game-container">
	<div class="svg-container">	
		<svg viewBox='0 0 100 100'>
			{#each boxes as row, j}
				{#each row as box, i}
					<rect x={i * spaceBetween + offset} y={j * spaceBetween + offset} width={spaceBetween} height={spaceBetween} class:rect1={box === 1} class:rect2={box === -1} />
				{/each}
			{/each}

			{#each dotsVertical as row}
				{#each dotsHorizontal as col}
					<circle r={$radiusSpring} cx="{spaceBetween * col + offset}" cy="{spaceBetween * row + offset}" />
				{/each}
			{/each}

			{#if linesVisible}
				{#each horizontalLines as lines, row}
					{#each lines as _, col}
						<line
							in:draw="{{duration: 1000, delay: 150 * row}}"
							out:draw="{{duration: 300}}"
							style="stroke-width: {lineWidth};"
							on:click="{() => handleHorizontalLineClick(row, col)}"
							class:not-selected={horizontalLines[row][col] === 0}
							class:player1={horizontalLines[row][col] === +1}
							class:player2={horizontalLines[row][col] === -1}
							x1="{offset + (col * spaceBetween) + radius}"
							y1="{offset + (row * spaceBetween)}" 
							x2="{offset + ((col + 1) * spaceBetween) - radius}" 
							y2="{offset + (row * spaceBetween)}"
						/>
					{/each}
				{/each}

				{#each verticalLines as lines, row}
					{#each lines as _, col}
						<line
							in:draw="{{duration: 1000, delay: 150 * row}}"
							out:draw="{{duration: 300}}"
							style="stroke-width: {lineWidth};"
							on:click="{() => handleVerticalLineClick(row, col)}"
							class:not-selected={verticalLines[row][col] === 0}
							class:player1={verticalLines[row][col] === 1}
							class:player2={verticalLines[row][col] === -1}
							x1="{offset + (row * spaceBetween)}"
							y1="{offset + (col * spaceBetween) + radius}" 
							x2="{offset + (row * spaceBetween)}" 
							y2="{offset + ((col + 1) * spaceBetween) - radius}"
						/>
					{/each}
				{/each}
			{/if}
		</svg>
	</div>

	<div class="controls">
		<label>
			<p>circle radius</p>
			<input type=range min=0.1 max=5 bind:value={radius} />
		</label>
		
		<label>
			<p>line width</p>
			<input type=range min=0.1 max=5 bind:value={lineWidth} />
		</label>
		
		<label>
			<p>width</p>
			<input type="number" bind:value={dotsHorizontalCount} />
		</label>
		
		<label>
			<p>height</p>
			<input type="number" bind:value={dotsVerticalCount} />
		</label>
		<button on:click={resetGame}>Reset game</button>
	</div>
</div>

<footer>
	Hamza Muric - Dots and Boxes
</footer>

{#if showModal}
	<Modal 
		winner={player === 1 ? 'Player 1' : 'Player 2'} 
		on:close="{() => showModal = false}"
		on:reset={resetGame}
	/>
{/if}


<style>
	:global(body) {
		background-color: whitesmoke;
		padding: 0;
	}
	.game-container {
		display: flex;
		justify-content: space-evenly;
	}
	.svg-container {
		max-width: 600px;
		/* margin: 20px auto; */
	}
	svg {
		background-color: white;
		border-radius: 10px;
		box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
	}
	.not-selected {
		stroke: lightgrey;
	}
	.not-selected:hover {
		stroke: grey;
		cursor: pointer;
	}
	.player1 {
		stroke: #0050b3;
	}
	.player2 {
		stroke: #f5222d;
	}
	.controls {
		background-color: white;
		max-width: 600px;
		/* margin: auto auto 20px auto; */
		border-radius: 10px;
		box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
		padding: 20px;
		box-sizing: border-box;
		display: grid;
		grid-template-columns: 1fr 1fr;
	}
	footer {
		background-color: lightgrey;
		color: black;
		font-weight: bold;
		font-size: 2em;
		padding: 20px;
		text-align: center;
	}
	@media only screen and (max-width: 600px) {
		.controls {
			grid-template-columns: 1fr;
		}
	}
	h1 {
		margin: 20px auto;
		text-align: center;
	}
	.score {
		text-align: center;
		font-size: 2em;
		font-weight: 400;
	}
	.score1 {
		color: #0050b3;
	}
	.score2 {
		color: #f5222d;
	}
	rect {
		fill: transparent;
	}
	.rect1 {
		fill: #40a9ff;
	}
	.rect2 {
		fill: #ff4d4f;
	}
</style>
