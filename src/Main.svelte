<script>
	import Welcome from './Welcome.svelte';
	import App from './App.svelte';

	let page = 'welcome';
	let loadedGame;


	const MODE_HH = 0;

	let mode = MODE_HH;
	let difficulty1 = 0;
	let difficulty2 = 0;
	let depth1 = 2;
	let depth2 = 2;
	let width = 4;
	let height = 4;
</script>

{#if page === 'welcome'}
	<Welcome
		bind:mode={mode}
		bind:difficulty1={difficulty1}
		bind:difficulty2={difficulty2}
		bind:depth1={depth1}
		bind:depth2={depth2}
		bind:width={width}
		bind:height={height}
		bind:loadedGame={loadedGame}
		on:click="{() => page = "game"}" />
{:else if page === 'game'}
	<App
		mode={mode}
		difficulty1={difficulty1}
		difficulty2={difficulty2}
		depth1={depth1}
		depth2={depth2}
		bind:width={width}
		bind:height={height}
		bind:loadedGame={loadedGame}
		onClose="{() => page = "welcome"}" />
{/if}


<footer>
	Hamza Muric - Dots and Boxes
</footer>

<style>
	footer {
		width: 100%;
		background-color: lightgrey;
		color: black;
		font-weight: bold;
		font-size: 2em;
		padding: 20px;
		text-align: center;
	}
</style>
