<script>
  import { onMount } from 'svelte';
  import { Linkedin, Coffee } from 'lucide-svelte';

  let year = new Date().getFullYear();
  let hover = '';

  onMount(() => {
    const timer = setInterval(() => {
      year = new Date().getFullYear();
    }, 1000);

    return () => clearInterval(timer);
  });

  const iconStyle = "w-6 h-6 inline-block transition-transform duration-300 ease-in-out";
  const linkStyle = "font-semibold underline underline-offset-4 decoration-2 transition-colors duration-300";
</script>

<footer class="print:hidden text-stone-800 my-16 p-4 bg-gradient-to-r from-blue-100 to-purple-100 rounded-lg shadow-md">
  <div class="text-center space-y-4">
    <p class="text-xl font-bold">
      © {year} - Time flies when you're coding!
    </p>
    <div class="flex justify-center items-center">
      <a
        href="https://www.linkedin.com/in/uddalak-das-crypto-nft-defi/"
        target="_blank"
        rel="noopener noreferrer"
        class="{linkStyle} hover:text-indigo-600"
        on:mouseenter={() => hover = 'linkedin'}
        on:mouseleave={() => hover = ''}
      >
        <Linkedin class="{iconStyle} {hover === 'linkedin' ? 'rotate-12' : ''}" />
        <span class="ml-2">Uddalak Das</span>
      </a>
    </div>
    <p class="italic text-sm">
      Built with
      <Coffee 
        class="{iconStyle} mx-1 text-brown-600 {hover === 'coffee' ? 'animate-bounce' : ''}" 
        on:mouseenter={() => hover = 'coffee'}
        on:mouseleave={() => hover = ''}
      />
      and a lot of debugging!
    </p>
  </div>
</footer>

<style>
  /* Add any additional styles here if needed */
</style>
