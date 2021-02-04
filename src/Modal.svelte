<script>
    import { createEventDispatcher } from 'svelte';

	export let winner;

    const dispatch = createEventDispatcher();
	const close = () => dispatch('close');
	const reset = () => dispatch('reset');

</script>

<style>
    .modal-background {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0,0,0,0.3);
	}

	.modal {
		position: absolute;
		left: 50%;
		top: 50%;
		width: calc(100vw - 4em);
		max-width: 32em;
		max-height: calc(100vh - 4em);
		overflow: auto;
		transform: translate(-50%,-50%);
		padding: 1em;
		border-radius: 0.2em;
		background: white;
	}
	h1 {
		text-align: center;
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

<div class="modal-background" on:click={close}></div>

<div class="modal" role="dialog" aria-modal="true">
	<h1>{winner} WON!</h1>
	<hr>
	<div class="buttons">
		<button on:click={reset}>Reset</button>
		<button on:click={close}>Close</button>
	</div>
</div>