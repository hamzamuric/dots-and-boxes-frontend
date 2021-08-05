<svelte:head>
	<meta property="og:image" content="http://azer.io/assets/imgs/dotlinkapp-logo.png" />
</svelte:head>

<script>
	import { onMount, tick } from 'svelte';
	import { fade, draw, fly } from 'svelte/transition';
	import { spring } from 'svelte/motion';
	import axios from 'axios';
	import Modal from './Modal.svelte';
	import { stringToNumber, numberToString } from './utils';

	export let mode;
	export let difficulty1;
	export let difficulty2;
	export let depth1;
	export let depth2;
	export let width;
	export let height;
	export let loadedGame;
	export let onClose;

	const url = '/ai/';
	// const url = 'http://127.0.0.1:7878/';

	axios.defaults.allowCredentials = true;

	let linesVisible = false;

	let gameSaved = !!localStorage.getItem('saved game');

	onMount(() => {
		if (loadedGame) {
			const chosenMode = mode;
			mode = 0;
			width = loadedGame.width;
			height = loadedGame.height;
			linesVisible = true;
			for (const m of loadedGame.movesPlayed) {
				const code = m.code;
				if (code[0] >= '0' && code[0] <= '9') {
					handleHorizontalLineClick(+code[0], stringToNumber(code[1]));
				} else {
					handleVerticalLineClick(+code[1], stringToNumber(code[0]));
				}
			}
			mode = chosenMode;
			loadedGame = null;
		} else {
			linesVisible = true;
		}
		if (mode === 2 /* ai vs ai */) {
			startAIs();
		}
	});

	let lineWidth = 2;
	let radius = 3;
	let radiusSpring = spring(0);
	$: radiusSpring.set(linesVisible ? 6 : 0);

	
	let dotsHorizontalCount = width;
	let dotsVerticalCount = height;
	let player = 1;

	let clickEnabled = true;

	let movesPlayed = [];
	
	$: spaceBetween = 100 / Math.max(dotsHorizontalCount, dotsVerticalCount);
	$: offset = (100 - (Math.max(dotsHorizontalCount, dotsVerticalCount) - 1) * spaceBetween) / 2;
	
	$: horizontalLines = Array(dotsVerticalCount).fill()
		.map(() => Array(dotsHorizontalCount - 1).fill().map(() => 0));

	$: verticalLines = Array(dotsHorizontalCount).fill()
		.map(() => Array(dotsVerticalCount - 1).fill().map(() => 0));
	
	$: dotsHorizontal = Array(dotsHorizontalCount).fill().map((_, i) => i);
	$: dotsVertical = Array(dotsVerticalCount).fill().map((_, i) => i);

	$: boxes = Array(dotsVerticalCount - 1).fill().map(() => Array(dotsHorizontalCount - 1).fill(0));
	$: console.debug(boxes);
	$: blueBoxes = boxes.flat().filter(b => b == 1).length;
	$: redBoxes = boxes.flat().filter(b => b == -1).length;

	$: win = ![...horizontalLines.flatMap(x => x), ...verticalLines.flatMap(x => x)].some(x => x === 0);

	let showModal = false;
	
	async function fillBoxes() {
		let filled = false;
		for (let i = 0; i < dotsVerticalCount - 1; i++) {
            for (let j = 0; j < dotsHorizontalCount - 1; j++) {
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
		if (mode === 0) return;
		if (player === 1 && mode === 1) return;
		const data = {
			horizontalLines,
			verticalLines,
			difficulty: player === 1 ? difficulty1 : difficulty2,
			depth: player === 1 ? depth1 : depth2,
		};
		try {
			clickEnabled = false;
			const response = await axios.post(url, data);
			clickEnabled = true;
			console.log('RESPONSE', response.data);
			if (!response.data.error) {
				const { side, i, j } = response.data;
				if (side === 0) {
					handleHorizontalLineClick(i, j);
				} else {
					handleVerticalLineClick(i, j);
				}
			}
		} catch (e) {
			console.log('ERRORRRRRRRR');
			console.log(Object.keys(e), e.response);
			clickEnabled = true;
		}
	}

	function handleHorizontalLineClick(row, col) {
		if (!clickEnabled) return;
		if (horizontalLines[row][col] !== 0) return;
		horizontalLines[row][col] = player;
		movesPlayed = [...movesPlayed, { player, code: `${row}${numberToString(col)}` }];
		fillBoxes();
		if (mode > 0) {
			sendToServer();
		}
	}
	
	function handleVerticalLineClick(row, col) {
		if (!clickEnabled) return;
		if (verticalLines[row][col] !== 0) return;
		verticalLines[row][col] = player;
		movesPlayed = [...movesPlayed, { player, code: `${numberToString(col)}${row}` }]
		fillBoxes();
		if (mode > 0) {
			sendToServer();
		}
	}

	function resetGame() {
		boxes = boxes.map(list => list.map(() => 0));
		movesPlayed = [];
		linesVisible = false;
		setTimeout(() => {
			horizontalLines = horizontalLines.map(lines => lines.map(() => 0));
			verticalLines = verticalLines.map(lines => lines.map(() => 0));
			player = 1;
			linesVisible = true;
			clickEnabled = true;
		}, 800);
		if (mode === 2 /* ai vs ai */) {
			startAIs();
		}
	}

	function saveToFile() {
		const game = {
			width,
			height,
			movesPlayed
		};
		localStorage.setItem('saved game', JSON.stringify(game));
		gameSaved = true;
	}

	function clearSaved() {
		localStorage.removeItem('saved game');
		gameSaved = false;
	}

	function startAIs() {
		setTimeout(() => {
			sendToServer();
		}, 2000);
	}
</script>

<div class="exit" on:click="{onClose}">
	<i class="nes-icon close is-large nes-pointer"></i>
</div>

<div class="game-container">
	<div class="score-container">
		<h1 style="color: {player === 1 ? '#0050b3' : '#f5222d'};">
			{player === 1 ? 'Player 1' : 'Player 2'}
		</h1>
		<div class="score">
			<span class="score1">{blueBoxes}</span> | <span class="score2">{redBoxes}</span>
		</div>
	</div>
	<div class="controls">
		<button type="button" class="nes-btn is-warning" on:click={resetGame}>Reset game</button>
		<button type="button" class="nes-btn is-primary" on:click={saveToFile}>Save game</button>
		{#if gameSaved}
			<button type="button" class="nes-btn is-error" on:click={clearSaved}>Clear saved</button>
		{/if}
	</div>

	<div class="svg-container nes-container is-rounded">	
		<svg viewBox='0 0 100 100'>
			{#each boxes as row, j}
				{#each row as box, i}
					{#if box != 0}
						<rect 
							in:fade
							out:fade="{{duration: 300, delay: 400 - ((i + j) * 100)}}" 
							x={i * spaceBetween + offset} y={j * spaceBetween + offset} width={spaceBetween} height={spaceBetween} class:rect1={box === 1} class:rect2={box === -1} />
					{/if}
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
							class="nes-pointer"
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
							class="nes-pointer"
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

			{#each dotsVertical as row}
				{#each dotsHorizontal as col}
					<!-- <circle r={$radiusSpring} cx="{spaceBetween * col + offset}" cy="{spaceBetween * row + offset}" /> -->
					<rect x="{spaceBetween * col + offset - $radiusSpring / 2}" y="{spaceBetween * row + offset - $radiusSpring / 2}" width="{$radiusSpring}" height="{$radiusSpring}" />
				{/each}
			{/each}
		</svg>
	</div>

	<div class="moves nes-container with-title is-rounded is-centered">
		<p class="title">Moves</p>
		{#each movesPlayed as move}
			<span
				in:fly="{{ y: 200, duration: 300 }}"
				out:fly="{{ y: 200, duration: 800 }}"
				style="color: { move.player === 1 ? 'blue' : 'red' }">{move.code}</span>
		{/each}
	</div>
</div>

{#if showModal}
	<Modal 
		winner={blueBoxes == redBoxes ? 0 : blueBoxes > redBoxes ? 1 : -1} 
		on:click="{() => showModal = false}"
		on:reset={resetGame}
		bind:showModal={showModal}
	/>
{/if}


<style>
	* {
		margin-top: 10px;
	}
	:global(body) {
		background-color: whitesmoke;
		padding: 0;
	}
	.exit {
		position: absolute;
		top: 10px;
		right: 30px;
	}
	.game-container {
		padding: 20px;
		display: grid;
		row-gap: 20px;
		column-gap: 20px;
		grid-template-columns: 1fr 1fr;
		grid-template-rows: min-content min-content 1fr;
		grid-template-areas:
			"game score"
			"game controlls"
			"game moves";
	}
	@media only screen and (max-width: 700px) {
		.game-container {
			grid-template-columns: 1fr;
			grid-template-areas:
				"score"
				"controlls"
				"game"
				"moves";
		}
	}
	.moves {
		grid-area: moves;
	}
	.moves span {
		margin: 6px;
		float: left;
		font-size: 1.5rem;
	}
	.svg-container {
		width: 100%;
		grid-area: game;
	}
	.not-selected {
		stroke: lightgrey;
	}
	.not-selected:hover {
		stroke: grey;
	}
	.player1 {
		stroke: #0050b3;
	}
	.player2 {
		stroke: #f5222d;
	}
	.controls {
		width: 100%;
		padding: 10px;
		box-sizing: border-box;
		grid-area: controlls;
		display: flex;
		gap: 12px;
	}
	.controls * {
		margin: 4px;
		font-size: 0.8rem;
		flex: 1;
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
	.score-container {
		grid-area: score;
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
	.rect1 {
		fill: #40a9ff;
	}
	.rect2 {
		fill: #ff4d4f;
	}
</style>
