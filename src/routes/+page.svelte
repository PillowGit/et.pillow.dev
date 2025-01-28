<script>
	import { onMount } from "svelte";
	import { animate } from "motion";
	import appointments from "$lib/resources/appointments.json";
	import MotionFadeIn from "$lib/components/MotionFadeIn.svelte";

	// Time conversion functions
	// All times are stored as "epoch minutes from January 19th, 2025, 00:00",
	// which represents the start of the semester.
	function toEpoch(date) {
		const diff_in_ms = date.getTime() - new Date("2025-01-19T00:00:00-08:00").getTime();
		return Math.floor(diff_in_ms / (60 * 1000));
	}
	function toDate(epoch) {
		return new Date(new Date("2025-01-19T00:00:00-08:00").getTime() + epoch * 60 * 1000);
	}
	// Converts a date to a nicely formatted 00:00 am/pm string
	function toTimeString(date) {
		return `${(date.getHours() != 12) ? date.getHours()%12 : 12}:${(date.getMinutes() != 0) ? date.getMinutes() : '00'} ${(date.getHours() < 12) ? 'am' : 'pm'}`;
	}

	// Stores all calendar info for this week
	let week = Math.floor(toEpoch(new Date()) / (168*60)) + 1;
	const week_min_start = (week - 1) * 168 * 60;
	let calendar_days = ['mon', 'tue', 'wed', 'thu', 'fri'];
	const calendar_min_offsets = {
		'mon': 1440,
		'tue': 1440*2,
		'wed': 1440*3,
		'thu': 1440*4,
		'fri': 1440*5
	};
	let calendar = {
		'week': week,
		'mon': {
			'date': toDate(calendar_min_offsets['mon'] + week_min_start),
			'appointments': []
		},
		'tue': {
			'date': toDate(calendar_min_offsets['tue'] + week_min_start),
			'appointments': []
		},
		'wed': {
			'date': toDate(calendar_min_offsets['wed'] + week_min_start),
			'appointments': []
		},
		'thu': {
			'date': toDate(calendar_min_offsets['thu'] + week_min_start),
			'appointments': []
		},
		'fri': {
			'date': toDate(calendar_min_offsets['fri'] + week_min_start),
			'appointments': []
		}
	}

	// Function that does all of the loading and css for changing the week
	async function displayWeek(week_num) {
		const week_hour = (week_num - 1) * 168;
		const week_min_start = week_hour * 60; // note that this is sun 00:00
		const week_apts = appointments[week_hour];
		// Clear all old elements
		let last_promise = null;
		for (let day of calendar_days) {
			for (let i = 0; i < calendar[day]['appointments'].length; i++) {
				if (calendar[day]['appointments'][i].filler) continue;
				const elem = document.getElementById(calendar[day]['appointments'][i].start);
				if (elem) {
					last_promise = animate(elem, {
						opacity: [1, 0],
						y: [0, 10],
						duration: 0.5
					}).then(() => {
						elem.remove();
					});
				}
			}
		}
		if (last_promise) await last_promise;
		for (let day of calendar_days) {
			calendar[day]['appointments'] = [];
		}
		calendar = { ...calendar };
		console.log(JSON.stringify(calendar['mon']['appointments']));
		// Add new elements
		for (let day of calendar_days) {
			const day_at_9_am = calendar_min_offsets[day] + week_min_start + 9*60;
			const day_at_8_pm = calendar_min_offsets[day] + week_min_start + 20*60;
			const apts = week_apts.filter((apt) => apt['start'] >= day_at_9_am && apt['start'] <= day_at_8_pm);
			let css = "";
			if (day != 'fri') {
				css += "border-right: 2px solid #f5f5f528; border-right-width: 2px; ";
			}
			css += "grid-template-columns: 1fr; height: 100%; width: 100%; display: grid; grid-template-rows: ";

			let i = day_at_9_am;
			let cur_gap = 0;
			while (i < day_at_8_pm) {
				const cur_apt = apts.filter((apt) => apt['start'] == i);
				if (cur_apt.length == 0) {
					cur_gap++;
				} else {
					if (cur_gap > 0) {
						calendar[day]['appointments'].push({
							'filler': true,
							'amnt': cur_gap,
							'last': false
						});
						css += `repeat(${cur_gap}, 1fr) `;
						cur_gap = 0;
					}
					const time_diff = cur_apt[0]['end'] - cur_apt[0]['start'];
					const segments = Math.floor(time_diff / 30);
					css += `${segments}fr `;
					calendar[day]['appointments'].push({
						'filler': false,
						'last': false,
						'start': cur_apt[0]['start'],
						'end': cur_apt[0]['end'],
						'title': cur_apt[0]['title'],
						'in-person': cur_apt[0]['in-person'],
						'zoom-link': cur_apt[0]['zoom-link']
					});
					i += time_diff - 30;
				}
				i += 30;
			}
			if (cur_gap > 0) {
				calendar[day]['appointments'].push({
					'filler': true,
					'amnt': cur_gap,
					'last': true
				});
				css += `repeat(${cur_gap}, 1fr) `;
			}
			css += ";";
			const week_elem = document.getElementById(day);
			week_elem.setAttribute("style", css);
			calendar['week'] = week_num;
			calendar[day]['date'] = toDate(calendar_min_offsets[day] + week_min_start);
		}
		calendar = {...calendar};
		console.log(JSON.stringify(calendar['mon']['appointments']));
	}

	// Change the week, both in variable and in calendar
	let lock = false;
	async function changeWeek(week_num) {
		if (lock) return;
		lock = true;
		await displayWeek(week_num);
		week = week_num;
		lock = false;
	}

	onMount(() => {
		// todo: Calculate week, call week change function
		changeWeek(week);
	});
</script>

<title>ET Schedule</title>
<!-- Colors from https://www.pinterest.com/pin/323133342032115696/ -->
<div
	class="flex min-h-screen w-screen min-w-[325px] flex-col items-center justify-center bg-[#131313] text-[#F5F5F5] text-center"
>
	<!-- Calendar header (title & days/dates) -->
	<div class="flex flex-col items-center justify-between w-[80%] min-w-[300px] h-[17.5%] min-h-[125px]">
		<div class="flex w-full flex-row justify-center items-center mb-4">
			<div
				role="button"
				aria-pressed="false"
				tabindex="0"
				on:click={() => {
					if (week <= 1) return;
					changeWeek(week - 1);
				}}
				on:keydown={(e) => {
					if (e.key == "Enter" || e.key == " ") {
						if (week <= 1) return;
						changeWeek(week - 1);
					}
				}}
			>
				<img src="/icons/left_arrow.svg" alt="arrow left" class={
					(week <= 1)
					? "transition invert md:h-9 sm:h-6 opacity-50"
					: "transition invert md:h-9 sm:h-6 hover:opacity-80 hover:scale-110"
				}/>
			</div>
			<div class="md:text-4xl sm:text-2xl font-bold ml-2 mr-2">Tutoring Schedule Week {calendar['week']}</div>
			<div
				role="button"
				aria-pressed="false"
				tabindex="0"
				on:click={() => {
					if (week >= Object.keys(appointments).length) return;
					changeWeek(week + 1);
				}}
				on:keydown={(e) => {
					if (e.key == "Enter" || e.key == " ") {
						if (week >= Object.keys(appointments).length) return;
						changeWeek(week + 1);
					}
				}}
			>
				<img src="/icons/left_arrow.svg" alt="arrow right" class={
					(week >= Object.keys(appointments).length)
					? "transition rotate-180 invert md:h-9 sm:h-6 opacity-50"
					: "transition rotate-180 invert md:h-9 sm:h-6 hover:opacity-80 hover:scale-110"
				}/>
			</div>
		</div>
		<div class="grid grid-cols-5 w-full h-[70%]">
			{#each calendar_days as day}
				<div class={(day == 'fri')
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
	<div class="grid grid-cols-5 w-[80%] min-w-[300px] h-[62.5%] min-h-[425px] mt-8 border border-[#f5f5f5] rounded-md">
		{#each calendar_days as day}
			<!-- Week body -->
			<div id={day}>
				{#each calendar[day]['appointments'] as appointment}
					{#if appointment['filler']}
						<!-- Empty/filler times -->
						{#each Array(appointment['amnt']-1)}
							<div class="w-full h-full border-b-2 border-b-[#f5f5f528] border-dotted"></div>
						{/each}
						<div class={(appointment['last'])
						? "w-full h-full"
						: "w-full h-full border-b-2 border-b-[#f5f5f528] border-dotted"}></div>
					{:else}
						<!-- Actual appointments -->
						<MotionFadeIn selfId={appointment['start']}>
						<div class="h-full w-full rounded-md" id={appointment['start']}>
							<div class="h-full w-full bg-[#4A4E69] rounded-lg flex flex-col items-start justify-start">
								<h3 class="md:ml-2 mt-2 font-bold md:text-base text-xs">{appointment['title']}</h3>
								<p class="md:ml-2 mt-2 mb-2 md:text-sm text-xs">{toTimeString(toDate(appointment['start']))} - {toTimeString(toDate(appointment['end']))}</p>
								{#if appointment['in-person']}
									<p class="md:ml-2 mt-2 mb-2 md:text-sm text-xs">In Person @ ULC</p>
								{:else}
									<p class="md:ml-2 mt-2 mb-2 md:text-sm text-xs text-wrap"><a href={appointment['zoom-link']} class="underline">Zoom meeting</a></p>
								{/if}
							</div>
						</div>
						</MotionFadeIn>
					{/if}
				{/each}
			</div>
		{/each}
	</div>
</div>
