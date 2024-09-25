<script>
	import { LogIn } from 'lucide-svelte';
	import { UserRoundSearch } from 'lucide-svelte';
	import { Code } from 'lucide-svelte';
	import FeatureItem from '$lib/components/FeatureItem.svelte';
	export let title = 'Enter Title Here';
	export let subtitle = 'Enter subtitle here';
</script>

<div class="print:hidden bg-[#FAFAFA] pt-8 pb-8 mb-2 border-b border-[#E2E2E2]">
	<div class="max-w-screen-md mx-auto flex flex-col items-center gap-4">
		<h1 class="text-center font-bold text-4xl text-[#1E6F5C]">
			{title}
		</h1>
		<p class="text-center text-[#333333] opacity-80">
			{subtitle}
		</p>
		<div class="flex flex-row">
			<FeatureItem label="No Signups.">
				<LogIn color="#1E6F5C" />
			</FeatureItem>
			<FeatureItem label="No Ads.">
				<UserRoundSearch color="#1E6F5C" />
			</FeatureItem>
			<FeatureItem label="Only what you need.">
				<Code color="#1E6F5C" />
			</FeatureItem>
		</div>
	</div>
</div>
