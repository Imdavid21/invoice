<script>
	import '../app.css';
	import Header from '$lib/components/Header.svelte';
	import Footer from '$lib/components/Footer.svelte';

	// Supports weights 100-900
	import '@fontsource-variable/inter';
</script>

<Header
	title="Zero Commitments. No Strings Attached."
	subtitle="Create and download invoices instantly, with zero signups or tracking."
/>
<slot />
<Footer repo="billie" />
