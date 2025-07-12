<script lang="ts">
	import { Temporal } from "temporal-polyfill";

	let setsGoal = $state(15);
	let timeGoal = $state(30);
	let prompt = $state("");

	let timerState = $state({
		started: false,
		secondsLeft: 0,
		roundsFinished: 0,
	});

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

	let timerId: number | undefined = undefined;

	const handleClick = () => {
		timerState = {
			...timerState,
			started: true,
			secondsLeft: setTime.minutes * 60 + setTime.seconds,
		};
		prompt = "running...";
		if (timerId) {
			clearInterval(timerId);
		}
		timerId = setInterval(() => {
			timerState = {
				...timerState,
				secondsLeft: timerState.secondsLeft - 1,
			};

			if (timerState.secondsLeft === 0) {
				timerState = {
					started: true,
					secondsLeft: setTime.minutes * 60 + setTime.seconds,
					roundsFinished: timerState.roundsFinished + 1,
				};

				if (timerState.roundsFinished == setsGoal) {
					prompt = "done!";
					clearInterval(timerId);
				}
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
	<span class="hl">{setTime.minutes}:{setTime.seconds}</span> per set (total: {totalTime}
	s).
</div>
<div>
	{prompt}
</div>
{#if timerState.started}
	<div id="running-countdown-info">
		<div>
			{Array.from(
				{ length: timerState.roundsFinished },
				(_) => "✅",
			).join("")}
			{#if timerState.roundsFinished > 0}
				{timerState.roundsFinished}
			{/if}
			{#if timerState.roundsFinished < setsGoal}
				🏋 ({timerState.roundsFinished + 1})
			{/if}
		</div>
		<div id="countdown-number">
			{timerState.secondsLeft}
		</div>
	</div>
{/if}

<style>
	div {
		font-size: 1.5em;
		font-family: Georgia, "Times New Roman", Times, serif;
	}

	#form {
		display: flex;
		width: 200px;
	}

	#countdown-number {
		font-size: 20em;
	}

	.hl {
		background-color: red;
		color: white;
	}
</style>
