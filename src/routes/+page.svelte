<script>
	import { onMount } from "svelte";
	import { animate } from "motion";
	import appointments from "$lib/resources/appointments.json";

	// Time conversion functions
	// All times are stored as "epoch minutes from January 19th, 2025, 00:00",
	// which represents the start of the semester.
	function toEpoch(date) {
		const diff = date.getTime() - new Date("2025-01-19T00:00:00Z").getTime();
		return Math.floor(diff / 60000);
	}
	function toDate(epoch) {
		return new Date(new Date("2025-01-19T00:00:00Z").getTime() + epoch * 60000);
	}

	// Stores all calendar info for this week
	const week = Math.floor(toEpoch(new Date()) / (168*60)) + 1;
	const week_min_start = (week - 1) * 168 * 60;
	const calendar_days = ['mon', 'tue', 'wed', 'thu', 'fri'];
	let calendar = {
		'week': week,
		'mon': {
			'date': toDate(2880 + week_min_start),
			'appointments': []
		},
		'tue': {
			'date': toDate(4320 + week_min_start),
			'appointments': []
		},
		'wed': {
			'date': toDate(5760 + week_min_start),
			'appointments': []
		},
		'thu': {
			'date': toDate(7200 + week_min_start),
			'appointments': []
		},
		'fri': {
			'date': toDate(8640 + week_min_start),
			'appointments': []
		}
	}

	onMount(() => {
		// todo: Calculate week, call week change function
		console.log(appointments);
	});
</script>

<title>ET Schedule</title>
<!-- Colors from https://www.pinterest.com/pin/323133342032115696/ -->
<div
	class="flex min-h-screen w-screen flex-col items-center justify-center bg-[#131313] text-[#F5F5F5]"
>
	<!-- Calendar header (title & days/dates) -->
	<div class="flex flex-col items-center justify-between w-[80%] min-w-[300px] h-[17.5%] min-h-[125px]">
		<h1 class="text-4xl font-bold">Tutoring Schedule Week {calendar['week']}</h1>
		<div class="grid grid-cols-5 w-full h-[70%]">
			{#each calendar_days as day}
				<div class={ day == 'fri'
				? "flex flex-col items-center justify-center w-full h-full"
				: "flex flex-col items-center justify-center w-full h-full border-[#f5f5f55b] border-r-[1px]"}>
					<p class="text-lg font-bold">{day.toLowerCase().replace(/(?:^|\s)\w/g, (match) => 
						 match.toUpperCase()
					)}</p>
					<p class="text-lg">{`${calendar[day]['date'].getMonth() + 1}/${calendar[day]['date'].getDate()}`}</p>
				</div>
			{/each}
		</div>
	</div>
	<!-- Calendar body -->
	<div class="flex flex-col items-center justify-center w-[80%] min-w-[300px] h-[62.5%] min-h-[425px]">
	</div>
</div>
