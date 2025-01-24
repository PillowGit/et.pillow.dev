<script>
	import { onMount } from 'svelte';
	import { animate } from 'motion';
	import appointments from '$lib/resources/appointments.json';

	// Time conversion functions
	// All times are stored as "epoch minutes from January 19th, 2025, 00:00",
	// which represents the start of the semester.
	function toEpoch(date) {
		const diff = date.getTime() - new Date('2025-01-19T00:00:00Z').getTime();
		return Math.floor(diff / 60000);
	}
	function toDate(epoch) {
		return new Date(new Date('2025-01-19T00:00:00Z').getTime() + epoch * 60000);
	}

	onMount(() => {
		// todo: Calculate week, call week change function
		console.log(appointments);
	});
</script>

<!-- Colors from https://www.pinterest.com/pin/323133342032115696/ -->
<div
	class="flex h-screen w-screen flex-col items-center justify-center bg-[#0A0A0A] text-[#F5F5F5]"
>
	<button
		class="mt-8 rounded-lg border-2 border-[#404242] bg-[#2a2e2a] transition-all hover:scale-105 hover:brightness-110"
		aria-pressed="false"
		tabindex="0"
		on:click={() => {
			const todays_date = new Date();
			alert(toEpoch(todays_date));
		}}
	>
		<p class="mb-2 ml-4 mr-4 mt-2">test</p>
	</button>
</div>
