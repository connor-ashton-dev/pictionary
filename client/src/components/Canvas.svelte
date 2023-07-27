<script lang="ts">
	import { onMount } from 'svelte';

	export let width = 1000;
	export let height = 1000;
	export let color = 'white';
	export let background = 'black';

	let canvas: HTMLCanvasElement;
	let context: CanvasRenderingContext2D | null;
	let isDrawing: boolean;
	let start: { x: number; y: number };

	let t: number, l: number;

	onMount(() => {
		context = canvas.getContext('2d');
		if (!context) return;
		context.lineWidth = 3;

		handleSize();
	});

	$: if (context) {
		context.strokeStyle = color;
	}

	const handleStart = ({ offsetX: x, offsetY: y }: { offsetX: number; offsetY: number }) => {
		if (color === background) {
			if (!context) return;
			context.clearRect(0, 0, width, height);
		} else {
			isDrawing = true;
			start = { x, y };
		}
	};

	const handleEnd = () => {
		isDrawing = false;
	};
	const handleMove = ({ offsetX: x1, offsetY: y1 }: { offsetX: number; offsetY: number }) => {
		if (!isDrawing) return;
		if (!context) return;

		const { x, y } = start;
		context.beginPath();
		context.moveTo(x, y);
		context.lineTo(x1, y1);
		context.closePath();
		context.stroke();

		start = { x: x1, y: y1 };
	};

	const handleSize = () => {
		const { top, left } = canvas.getBoundingClientRect();
		t = top;
		l = left;
	};

	const handleClear = () => {
		if (!context) return;
		context.clearRect(0, 0, width, height);
	};
</script>

<svelte:window on:resize={handleSize} />

<canvas
	{width}
	{height}
	style:background
	bind:this={canvas}
	on:mousedown={handleStart}
	on:touchstart={(e) => {
		const { clientX, clientY } = e.touches[0];
		handleStart({
			offsetX: clientX - l,
			offsetY: clientY - t
		});
	}}
	on:mouseup={handleEnd}
	on:touchend={handleEnd}
	on:mouseleave={handleEnd}
	on:mousemove={handleMove}
	on:touchmove={(e) => {
		const { clientX, clientY } = e.touches[0];
		handleMove({
			offsetX: clientX - l,
			offsetY: clientY - t
		});
	}}
/>

<button on:click={handleClear}>Clear</button>
