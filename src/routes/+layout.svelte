<script>
	import '../app.css';
	import { page } from '$app/stores';
</script>

<svelte:head>
	<title>Svalgo</title>
</svelte:head>

<div class="flex flex-col justify-between items-center w-full h-screen py-4">
	<slot />
	<p>Made with ♥️ by <a href="https;//github.com/einKnuffy">einKnuffy</a></p>
</div>
