<script lang="ts">
	import { Temporal } from "temporal-polyfill";

	let setsGoal = $state(15);
	let timeGoal = $state(30);
	let prompt = $state("");

	const startTimer = (setTime: Temporal.Duration) => {};

	const computeSetTime = (timeGoalNumber: number, setsGoalNumber: number) => {
		const timeGoal = Temporal.Duration.from({ minutes: timeGoalNumber });
		const timeInSeconds = 60 * timeGoal.minutes + timeGoal.seconds;

		const secondsPerSet = Math.floor(timeInSeconds / setsGoalNumber);
		return Temporal.Duration.from({
			minutes: Math.floor(secondsPerSet / 60),
			seconds: secondsPerSet % 60,
		});
	};

	let setTime = $derived(computeSetTime(timeGoal, setsGoal));
	let totalTime = $derived(
		setTime.minutes * 60 * setsGoal + setTime.seconds * setsGoal,
	);

	let tick = $state(0);
	let countdownSeconds = $state(0);
	let countdownStarted = $state(false);

	const handleClick = () => {
		prompt = "starting the timer.";
		countdownStarted = true;
		countdownSeconds = setTime.minutes * 60 + setTime.seconds;
		setInterval(() => {
			tick = tick + 1;
			countdownSeconds -= 1;
		}, 1000);
	};
</script>

<input bind:value={setsGoal} />
<button on:click={handleClick}> start </button>

<div>
	{setsGoal} sets in {timeGoal} minutes. <br />
	{setTime.minutes}:{setTime.seconds} per set. <br />
	{totalTime} total.
</div>
<div>
	{prompt}
</div>
{#if countdownStarted}
	<div>
		{countdownSeconds}
	</div>
{/if}
