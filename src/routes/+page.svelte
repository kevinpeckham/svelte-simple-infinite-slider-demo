<!-- Example Svelte Page / Starter Web Page-->
<script lang="ts">
	// settings
	const durationSetting = 0.2; // keep it nice and snappy .15 - 3 seconds
	const panelWidth = 200;
	$: hideInactivePanels = false; // flip to false to see what's happening outside the frame

	// types
	interface Datum {
		color: string;
		content: string;
	}

	// data
	const data: Datum[] = [
		{ color: "red", content: "A" },
		{ color: "blue", content: "B" },
		{ color: "yellow", content: "C" },
		{ color: "green", content: "D" },
	];

	// animation duration reactive variable
	$: duration = durationSetting;

	// position of sliding container
	$: slidingContainerXPosition = panelWidth * -1;

	// set active panel
	$: active = data[1];

	// get index of active panel -- will update when active changes
	$: activeIndex = data.indexOf(active);

	// get previous -- will update when activeIndex changes
	$: previousIndex = activeIndex - 1 >= 0 ? activeIndex - 1 : data.length - 1;
	$: previous = data[previousIndex];

	// get next -- will update when activeIndex changes
	$: nextIndex = activeIndex + 1 < data.length ? activeIndex + 1 : 0;
	$: next = data[nextIndex]; // e.g. 'yellow'

	// an array of panels centered on the active panel -- note: needs to be manually updated when contents change
	$: panels = [previous, active, next];

	// sliding container ref
	let slidingContainer: HTMLDivElement;

	// panel refs
	let firstEl: HTMLDivElement;
	let secondEl: HTMLDivElement;
	let thirdEl: HTMLDivElement;
	let fourthEl: HTMLDivElement;
	let panelRefs: HTMLDivElement[];
	$: panelRefs = [firstEl, secondEl, thirdEl, fourthEl];

	// button refs
	let prevButton: HTMLButtonElement;
	let nextButton: HTMLButtonElement;
	$: buttonRefs = [prevButton, nextButton];

	// let timeout: NodeJS.Timeout;

	// functions
	function prevClick(event: MouseEvent) {
		// get reference to the button itself
		const target = event.target as HTMLButtonElement;

		// disable the button so it isn't clicked again until the animation is complete
		target.disabled = true;

		// move the sliding container to the right
		slidingContainerXPosition = slidingContainerXPosition + panelWidth;

		// wait till the animation is complete and then:
		setTimeout(() => {
			// add the next panel to the beginning of the array
			panels = [next, ...panels];

			// update the active panel
			active = panels[1];

			// change animation duration to 0
			duration = 0;

			// move the sliding container back to the center
			slidingContainerXPosition = slidingContainerXPosition - panelWidth;
		}, durationSetting * 1000);
		// return the transition duration default
		duration = durationSetting;
		// re-enable the button
		target.disabled = false;
	}
	function nextClick(event: MouseEvent) {
		// get reference to the button itself
		const target = event.target as HTMLButtonElement;

		// disable the button so it isn't clicked again until the animation is complete
		target.disabled = true;

		// add the previous panel to the end of the array
		panels = [...panels, previous];

		// move the sliding container to the left
		slidingContainerXPosition = slidingContainerXPosition - panelWidth;

		// wait till the animation is complete and then:
		setTimeout(() => {
			// remove extra panel from end of array
			panels = panels.slice(1);

			// change animation duration to 0
			duration = 0;

			// move the sliding container back to the center
			slidingContainerXPosition = slidingContainerXPosition + panelWidth;

			// update active panel
			active = panels[1];
		}, durationSetting * 1000);
		// return the transition duration default
		duration = durationSetting;
		// re-enable the button
		target.disabled = false;
	}
	const clickFunctions = [prevClick, nextClick];

	// style functions
	$: outerContainerStyle = `width: ${panelWidth}px;`;

	$: slidingContainerStyle = `
		left:0px;
		transition-duration: ${duration}s;
		transform: translateX(${slidingContainerXPosition}px);`;

	// let tailwind find these classes
	const twSafe = ["bg-yellow-600", "bg-red-600", "bg-blue-600", "bg-green-600"];

	// variables used in template below
	// defining here so we can add types
	let index: number;
	let panel: Datum;
	let color: string;
	let opacity: number;
	let pointerEvents: string;
	let label: string;
	let lt: string;
	let rt: string;
</script>

<template lang="pug">
	//- page body
	main.flex.items-center.justify-center.w-screen.h-screen
		//- page title
		h1.absolute.top-8.left-8.text-18 Svelte Simple Infinite Slider Demo

		//- outer container
		.relative.aspect-square(
			style!="{ outerContainerStyle }"
		)
			//- frame
			.outline.outline-white.relative.w-full.h-full(
				class!="{ hideInactivePanels ? 'overflow-hidden' : '' }"
			)
				//- sliding container to hold panels
				.absolute.flex.transition-transform.left-0(
					bind:this!="{ slidingContainer }",
					style!="{ slidingContainerStyle }"
				)
					//- panels
					//- you would probably want to make the panels their own component and pass in the data for most use cases
					//- but it's easier to show how it all works in a single file
					+each('panels as panel, index')
						+const('color = panel.color == "yellow" ? "text-primary" : "white"')
						+const('opacity = active == panel ? "opacity-100" : "opacity-60"')
						+const('bgColor = "bg-" + panel.color + "-600"')
						+const('pointerEvents = active == panel ? "pointer-events-auto" : "pointer-events-none"')
						+const('width = "width:" + panelWidth + "px"')
						.panel.transition-opacity.aspect-square.flex.items-center.justify-center(
							class!="{color} {opacity} {bgColor} {pointerEvents}",
							bind:this!="{ panelRefs[index] }",
							style!="{ width }"
						) { panel.content }

			//- buttons
			.translate-y-12.flex.gap-4.justify-center
				+each('["prev", "next"] as label, index')
					+const('lt = label == "prev" ? "&lt;&nbsp;" : ""')
					+const('gt = label == "next" ? "&nbsp;&gt;" : ""')
					button.rounded.outline.whitespace-nowrap.px-2.opacity-80(
						class="hover:opacity-100",
						bind:this!="{ buttonRefs[index] }",
						on:click!="{ clickFunctions[index] }"
					)
						+html('lt')
						| { label }
						+html('gt')

			//- toggle
			.flex.absolute.-bottom-32.left-0.z-10.text-center.justify-center.w-full.opacity-90.text-15
				.flex.gap-x-2
					input(
						bind:checked!="{ hideInactivePanels }",
						id="toggle",
						type="checkbox"
					)
					label(for="toggle") hide overflow
</template>
