<script>
	import { onMount } from 'svelte';
	const anchorStyle = 'decoration-2 underline underline-offset-4 font-semibold ';

	let year = new Date().getFullYear();

	onMount(() => {
		const timer = setInterval(() => {
			year = new Date().getFullYear();
		}, 1000);

		return () => clearInterval(timer);
	});
</script>

<footer class="print:hidden text-stone-800 my-16">
	<p class="p-2 m-2 text-center">
		© {year} · Built by
		<a
			target="_blank"
			class="{anchorStyle} decoration-indigo-500 hover:text-indigo-600 transition-colors duration-300"
			href="https://www.linkedin.com/in/uddalak-das-crypto-nft-defi/">Uddalak Das</a
		>
	</p>
</footer>

<style>
	/* Add any additional styles here if needed */
</style>
