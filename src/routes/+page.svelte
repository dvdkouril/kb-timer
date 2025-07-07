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
	let timerId: number | undefined = undefined;
	let roundsFinished = $state("");

	const handleClick = () => {
		prompt = "starting the timer.";
		countdownStarted = true;
		countdownSeconds = setTime.minutes * 60 + setTime.seconds;
		roundsFinished = "";
		if (timerId) {
			clearInterval(timerId);
		}
		timerId = setInterval(() => {
			tick = tick + 1;
			countdownSeconds -= 1;

			if (countdownSeconds === 0) {
				roundsFinished += "✅";
				countdownSeconds = setTime.minutes * 60 + setTime.seconds;
			}
		}, 1000);
	};
</script>

<div id="form">
	<label>
		Sets:
		<input bind:value={setsGoal} />
	</label>
	<label>
		Time:
		<input bind:value={timeGoal} />
	</label>
	<button on:click={handleClick}> start </button>
</div>

<div>
	<span class="hl">{setsGoal}</span> sets in
	<span class="hl">{timeGoal}</span>
	minutes:
	<span class="hl">{setTime.minutes}:{setTime.seconds}</span> per set. ({totalTime}
	total.)
</div>
{#if countdownStarted}
	<div id="running-countdown-info">
		<div>
			{roundsFinished}
		</div>
		<div id="countdown-number">
			{countdownSeconds}
		</div>
	</div>
{/if}

<style>
	p,
	div {
		font-size: 1.5em;
		font-family: Georgia, "Times New Roman", Times, serif;
	}

	#form {
		display: flex;
		width: 200px;
	}

	#form {
	}
	#countdown-number {
		font-size: 20em;
	}

	.hl {
		background-color: red;
		color: white;
	}
</style>
