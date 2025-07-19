<script lang="ts">
	import { Button } from '$lib/components/ui/button';
	import { Progress } from '$lib/components/ui/progress';
	import * as Select from '$lib/components/ui/select/index';

	let FOCUS_TIME: number = 25 * 60;
	let BREAK_TIME = 5 * 60;

	let isBreakTimeActive: boolean = $state(false);
	let isTimerStarted: boolean = $state(false);
	let remainingTime: number = $state(FOCUS_TIME);
	let totalTimeInSeconds: number = $state(FOCUS_TIME);
	let timerInterval: number | undefined = $state(undefined);
	let triggerValue: number | string = $state(0);

	const startTimer = () => {
		timerInterval = setInterval(() => {
			remainingTime--;
		}, 1000);
		isTimerStarted = true;
	};

	const pauseTimer = () => {
		clearInterval(timerInterval);
		isTimerStarted = false;
	};

	const resetTimer = () => {
		remainingTime = FOCUS_TIME;
		totalTimeInSeconds = FOCUS_TIME;
		clearInterval(timerInterval);
		isTimerStarted = false;
		isBreakTimeActive = false;
	};

	const formatSeconds = (secs: number) => {
		const pad = (n: number) => (n < 10 ? `0${n}` : n);
		const m = Math.floor(secs / 60);
		const s = Math.floor(secs - m * 60);
		return `${pad(m)}:${pad(s)}`;
	};

	$effect(() => {
		if (remainingTime < 0 && !isBreakTimeActive) {
			isBreakTimeActive = true;
			BREAK_TIME = 300;
			clearInterval(timerInterval);
			isTimerStarted = false;
			remainingTime = BREAK_TIME;
			totalTimeInSeconds = BREAK_TIME;
		}
		if (remainingTime < 0 && isBreakTimeActive) {
			isBreakTimeActive = false;
			clearInterval(timerInterval);
			isTimerStarted = false;
			remainingTime = FOCUS_TIME;
			totalTimeInSeconds = FOCUS_TIME;
		}
	});
</script>

<p class="absolute left-1 rounded font-mono">
	POMO<span class="text-purple-500">doro</span>
</p>
<div
	class="mx-auto flex h-dvh max-w-full flex-col items-center justify-center gap-4 bg-zinc-100 p-6 dark:bg-zinc-900"
>
	<p class="text-center font-mono text-2xl font-bold text-purple-500">
		{isBreakTimeActive ? 'Take A Break🙇🏾‍♂️' : 'Focus👨🏾‍💻'}
	</p>
	<p
		class={`font-mono text-4xl font-bold ${remainingTime <= 60 ? 'text-red-500' : 'text-zinc-100'}`}
	>
		{formatSeconds(remainingTime)}
	</p>

	<Progress
		value={remainingTime}
		min={0}
		max={totalTimeInSeconds}
		class="h-4 w-full rounded bg-purple-500 [&>div]:bg-zinc-100"
	/>

	<Select.Root type="single">
		<Select.Trigger class="px-4"
			>{triggerValue === 0
				? 'Choose timer length'
				: `${Number(triggerValue) + 1 === 1 ? `${triggerValue} minute` : `${triggerValue} minutes`}`}</Select.Trigger
		>
		<Select.Content class="h-[300px]">
			<Select.Group>
				{#each { length: 25 }, minute}
					<Select.Item
						onclick={(event: MouseEvent) => {
							const value = (event.currentTarget as Element)?.getAttribute('data-value');
							if (value) {
								const focusMinutes = Number(value) + 1;
								triggerValue = focusMinutes;
								remainingTime = focusMinutes * 60;
								totalTimeInSeconds = remainingTime;
								FOCUS_TIME = focusMinutes * 60;
								isBreakTimeActive = false;
								isTimerStarted = false;
								clearInterval(timerInterval);
							}
						}}
						value={String(minute)}
						label={String(minute + 1)}
						>{minute + 1 === 1 ? `${minute + 1} minute` : `${minute + 1} minutes`}
					</Select.Item>
				{/each}
			</Select.Group>
		</Select.Content>
	</Select.Root>

	<div class="flex gap-2">
		<Button
			onclick={isTimerStarted ? pauseTimer : startTimer}
			class="rounded bg-purple-600 px-4 py-2 w-20 text-zinc-100 hover:bg-purple-700 dark:bg-purple-500 dark:hover:bg-purple-600"
		>
			{isTimerStarted ? 'Pause' : 'Start'}
		</Button>

		<Button
			onclick={resetTimer}
			class="rounded bg-zinc-600 px-4 py-2 w-20 text-zinc-100 hover:bg-zinc-700 dark:bg-zinc-500 dark:hover:bg-zinc-600"
		>
			Reset
		</Button>
	</div>
</div>
