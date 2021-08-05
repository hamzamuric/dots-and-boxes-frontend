<script>
	export let winner;
	export let showModal;
</script>

<style>
    .modal-background {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	.modal {
		position: absolute;
		left: 50%;
		top: 50%;
		display: inline-block;
		overflow: auto;
		transform: translate(-50%,-50%);
		padding: 1em;
		border-radius: 0.2em;
		background: white;
	}
	h1 {
		text-align: center;
		display: inline;
	}
	hr {
		width: calc(100% - 8px);
	}
	.buttons {
		display: flex;
	}
	.buttons button {
		flex: 1;
		margin: 3px;
	}
</style>

{#if showModal}
	<div
		class="modal-background"
		style="background-color: {winner === 0 ? 'transparent' : winner === 1 ? 'rgba(0, 0, 255, 0.3)' : 'rgba(255, 0, 0, 0.3)'}"
		on:click={() => {showModal = false}}
	></div>

	<div class="modal" role="dialog" aria-modal="true">
		<i class="nes-icon trophy is-large"></i>
		{#if winner === 0}
			<h1>It's a tie</h1>
		{:else}
			<h1>Player {winner === 1 ? 1 : 2} WON!</h1>
		{/if}
		<hr>
		<div class="buttons">
			<button type="button" class="nes-btn" on:click>Close</button>
		</div>
	</div>
{/if}
