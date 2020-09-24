<svelte:options tag="svelte-app" />

<script>
	export let name;
</script>

<main>
	<h1>Hello {name}!</h1>
</main>

<style>
	main {
		text-align: center;
		padding: 0.1rem;
		max-width: 100vw;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
	}

</style>
