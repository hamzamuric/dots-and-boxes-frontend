<script>
	import { createEventDispatcher } from "svelte";


	const MODE_HH = 0;
	const MODE_HAI = 1;
	const MODE_AIAI = 2;

	export let mode; 
	export let difficulty1;
	export let difficulty2;
	export let depth1;
	export let depth2;
	export let width;
	export let height;
	export let loadedGame;

	$: depth1 = difficulty1 * 2 + 2;
	$: depth2 = difficulty2 * 2 + 2;

	const dispatch = createEventDispatcher();

	function loadGame() {
		const savedGame = localStorage.getItem('saved game');
		const game = JSON.parse(savedGame);
		loadedGame = game;
		width = game.width;
		height = game.height;
		dispatch('click');
	}

	const hasSavedGame = !!localStorage.getItem('saved game');

</script>

<div class="page">
	<div class="content nes-container with-title is-rounded">
		<p class="title">Game preferences</p>
		<label for="mode-select">Mode</label>
		<div class="nes-select">
			<select id="mode-select" bind:value="{mode}">
				<option value="{MODE_HH}">Human - Human</option>
				<option value="{MODE_HAI}">Human - AI</option>
				<option value="{MODE_AIAI}">AI - AI</option>
			</select>
		</div>
		<br>

		
		<!-- depths class is from bottom div which is realy for depths -->
		<div class="depths">
			<div class="one-depth">
				<span>Width</span>
				<div class="counter-container nes-container is-rounded">
					<span>{width}</span>
					<button
						type="button" 
						class="nes-btn counter-btn"
						on:click="{() => width = width >= 2 && width < 7 ? width + 1 : width}"
					>+</button>
					<button
						type="button"
						class="nes-btn counter-btn"
						on:click="{() => width = width > 2 && width <= 7 ? width - 1 : width}"
					>-</button>
				</div>
			</div>

			<div class="one-depth">
				<span>Height</span>
				<div class="counter-container nes-container is-rounded">
					<span>{height}</span>
					<button
						type="button" 
						class="nes-btn counter-btn"
						on:click="{() => height = height >= 2 && height < 7 ? height + 1 : height}"
					>+</button>
					<button
						type="button"
						class="nes-btn counter-btn"
						on:click="{() => height = height > 2 && height <= 7 ? height - 1 : height}"
					>-</button>
				</div>
			</div>
		</div>
		<br>

		<div class="depths">
			{#if mode === MODE_AIAI}
				<div class="one-depth">
					<span>Depth 1</span>
					<div class="counter-container nes-container is-rounded">
						<span>{depth1}</span>
						<button
							type="button" 
							class="nes-btn counter-btn"
							on:click="{() => depth1 = depth1 >= 1 && depth1 < 7 ? depth1 + 1 : depth1}"
						>+</button>
						<button
							type="button"
							class="nes-btn counter-btn"
							on:click="{() => depth1 = depth1 > 1 && depth1 <= 7 ? depth1 - 1 : depth1}"
						>-</button>
					</div>
				</div>
			{/if}

			{#if mode > MODE_HH}
				<div class="one-depth">
					<span>Depth 2</span>
					<div class="counter-container nes-container is-rounded">
						<span>{depth2}</span>
						<button
							type="button" 
							class="nes-btn counter-btn"
							on:click="{() => depth2 = depth2 >= 1 && depth2 < 7 ? depth2 + 1 : depth2}"
						>+</button>
						<button
							type="button"
							class="nes-btn counter-btn"
							on:click="{() => depth2 = depth2 > 1 && depth2 <= 7 ? depth2 - 1 : depth2}"
						>-</button>
					</div>
				</div>
			{/if}
		</div>
		<br>

		{#if mode === MODE_AIAI}
			<p>Difficulty 1</p>
			<label>
				<input type="radio" class="nes-radio" name="answer1" bind:group="{difficulty1}" value="{0}" />
				<span>Easy</span>
			</label>
			<label>
				<input type="radio" class="nes-radio" name="answer1" bind:group="{difficulty1}" value="{1}" />
				<span>Medium</span>
			</label>
			<label>
				<input type="radio" class="nes-radio" name="answer1" bind:group="{difficulty1}" value="{2}" />
				<span>Hard</span>
			</label>
		{/if}
		{#if mode > MODE_HH}
			<p>Difficulty 2</p>
			<label>
				<input type="radio" class="nes-radio" name="answer2" bind:group="{difficulty2}" value="{0}" />
				<span>Easy</span>
			</label>
			<label>
				<input type="radio" class="nes-radio" name="answer2" bind:group="{difficulty2}" value="{1}" />
				<span>Medium</span>
			</label>
			<label>
				<input type="radio" class="nes-radio" name="answer2" bind:group="{difficulty2}" value="{2}" />
				<span>Hard</span>
			</label>
		{/if}
		<div style="height: 20px;"></div>
		<div class="buttons">
			<button 
				type="button" class="nes-btn is-primary"
				on:click
			>Play</button>
			{#if hasSavedGame}
				<button
					type="button" class="nes-btn"
					on:click="{loadGame}"
				>Load saved game</button>
			{/if}
		</div>
	</div>
</div>

<style>
	.page {
		width: 100%;
		height: 100vh;
		display: flex;
		margin-bottom: 100px;
	}
	.content {
		max-width: 600px;
		margin: auto;
	}
	.depths {
		display: flex;
		gap: 12px;
	}
	.depths div {
		flex: 1;
	}
	.counter-container {
		display: grid;
		grid-template-columns: 1fr min-content min-content;
		padding: 6px;
		place-items: center;
	}
	.counter-btn {
		margin: 0;
		padding: 4px;
	}
	.buttons {
		display: flex;
		justify-content: space-between;
	}
</style>
