<script>
	import '../app.css';
	import Header from '$lib/components/Header.svelte';
	import Footer from '$lib/components/Footer.svelte';

	// Supports weights 100-900
	import '@fontsource-variable/inter';
</script>

<Header
	title="Free Invoice Tool"
	subtitle="No signups. No commitments. Just quick, clean invoices."
/>
<slot />
<Footer repo="billie" />
