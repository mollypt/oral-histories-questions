<script>
	import { json } from "@sveltejs/kit";
	import data from "$data/narrative-time.csv"
	import metadata from "$data/interview-metadata.csv"
	import { onMount } from 'svelte';
	import { group, bisector } from "d3"
	

const TICK_WIDTH = .65
const TOPIC_MAPPING = {
    pearl_harbor: 24,
    belongings: 62,
    questionnaire: 72,
    hiroshima: 66,
    race: 13,
    food: 9
};

const COLOR_MAPPING = {
    24: "#c5e2dd",
    62: "#ffa27d",
    72: "#78f9ff",
    66: "#d5c4ee",
    13: "#1c3b50",
    9: "#ffd742"
}
const dataByTopic = group(data, d => d.topic);
let topicsInOrder = [24, 62, 72, 66, 13, 9]
dataByTopic.forEach(topicGroup => {
	topicGroup=topicGroup.sort((a,b)=>parseFloat(a.x)-parseFloat(b.x))
})
let timelineWidth = $state(0)
let timelineHeight = $state(0)
let hoveredTick = $state(null)
let hoveredTickQuestion = $state(null)
let hoveredTickAnswer = $state(null)

let staticTickHeight = $derived(timelineHeight * .6)
let onTickHeight = $derived(timelineHeight * .8)

let selectedTick = $state(null)
let tickFrozen = $state(false)
$inspect(tickFrozen)
let canvases = []

let narratorName = $derived(metadata.filter(m=> m.Filename == selectedTick?.file)[0].Narrator.replace(",", " & "))
let interviewerName = $derived(metadata.filter(m=> m.Filename == selectedTick?.file)[0].Interviewer.replace(",", " & "))
let interviewID = $derived(
    metadata
        .find(m => m.Filename == selectedTick?.file)
        ?.Filename.split("-transcript")[0]
);
let interviewURL = $derived(`https://ddr.densho.org/${interviewID}`)
const defaultText = "Each tick represents a question from an oral history in the Densho Digital Repository.<br><br>Hover on a tick to read the question and click to filter to the interview."
const defaultFilterText = $derived(`Select a tick to freeze the question.<br><br>Select the background to unfreeze or return to collection view.<br><br>View the full interview <a href="${interviewURL}" target="_blank">here</a>.`)
function findNearestTick(ticks, x) {
	if (selectedTick) {
		ticks = ticks.filter(t=>t.file === selectedTick.file)
	}
	if (!ticks.length) return null;

	// find where x pos would be inserted into tick list 
	const i = bisector(d => d.x).left(ticks, x);

	if (i === 0) {
		if (Math.abs(ticks[i].x - x) < .02) return ticks[0];
		else return null;
	} 
	if (i === ticks.length){
		if (Math.abs(ticks[i-1].x - x) > .02) return null;
		else return ticks[ticks.length - 1];
	}

	// return closest tick
	const distanceToRight = Math.abs(ticks[i].x - x)
	const distanceToLeft = Math.abs(ticks[i - 1].x - x)
	if (distanceToRight > .02 && distanceToLeft > .02) return null
    return distanceToRight <
        distanceToLeft
        ? ticks[i]
        : ticks[i - 1];
}

function drawCanvas(ctx, topic) {
	ctx.clearRect(0, 0, timelineWidth, timelineHeight);
	const topicData = dataByTopic.get(`${topic}`)

	for (const d of topicData) {
		let alpha = 1
		let lineWidth= TICK_WIDTH
		let tickHeight = staticTickHeight

		if (selectedTick) {
			if (d.file !== selectedTick.file) {
				alpha = .05
			} 
			else if (d.x === hoveredTick?.x) {
				tickHeight = onTickHeight*2
				lineWidth = TICK_WIDTH+4
			}
			else {
				lineWidth = TICK_WIDTH+2
				tickHeight = onTickHeight
			}
		} else if (hoveredTick) {
			if (d.x === hoveredTick.x) {
				lineWidth= TICK_WIDTH + 2
				tickHeight = onTickHeight
			}
		} 

		ctx.beginPath();
		const y1 = (timelineHeight - tickHeight) / 2 
		ctx.moveTo(d.x * timelineWidth, y1);
		ctx.lineTo(d.x * timelineWidth, y1+tickHeight);
		ctx.lineWidth = lineWidth;
		ctx.strokeStyle = COLOR_MAPPING[topic];
		ctx.globalAlpha = alpha;
		ctx.stroke();
	}
}
onMount(()=> {
	function drawAllCanvases() {
		canvases.forEach((canvas,i) => {
			const ctx = canvas.getContext("2d");
			resetDimensions(canvas, ctx)
			drawCanvas(ctx, topicsInOrder[i])
		})
	}

	function resetDimensions(canvas, ctx) {
		// dimensions
		const dpr = window.devicePixelRatio || 1;
		const pixelWidth = Math.round(timelineWidth * dpr);
		const pixelHeight = Math.round(timelineHeight * dpr);

		if (canvas.width !== pixelWidth ||
			canvas.height !== pixelHeight) {
			canvas.width = pixelWidth;
			canvas.height = pixelHeight;
		}
		ctx.setTransform(dpr, 0, 0, dpr, 0, 0);

	}
	canvases = Object.entries(TOPIC_MAPPING).map(([id, topic]) => {
		const canvas = document.getElementById(id);
		const ctx = canvas.getContext("2d");
	
		// dimensions
		resetDimensions(canvas, ctx)
		drawCanvas(ctx, topic);

		canvas.addEventListener("pointermove", event => {
			if (tickFrozen) return;
			const rect = canvas.getBoundingClientRect();
			const x = (event.clientX - rect.left) / rect.width;
			hoveredTick = findNearestTick(dataByTopic.get(`${topic}`), x);
			hoveredTickAnswer = hoveredTick?.answer;
			hoveredTickQuestion = hoveredTick?.question;
			drawCanvas(ctx, topic) 
		});

		canvas.addEventListener("pointerleave", () => {
			if (tickFrozen) return;
			hoveredTick = null;
			hoveredTickAnswer = null;
			hoveredTickQuestion = null;
			
			drawCanvas(ctx, topic) 
		});

		canvas.addEventListener("pointerdown", () => {
			if (tickFrozen) {
				tickFrozen = false;
				hoveredTick=null;
				hoveredTickAnswer = null;
				hoveredTickQuestion = null;
			}
			else if (!tickFrozen && hoveredTick && selectedTick) {
				tickFrozen = true
				selectedTick == hoveredTick
			}
			else if (selectedTick && hoveredTick?.file != selectedTick.file) {
				selectedTick = null;
				hoveredTick=null;
				hoveredTickAnswer = null;
				hoveredTickQuestion = null;
				tickFrozen =null;
			} else selectedTick = hoveredTick
			drawAllCanvases();
		});

		return canvas
	})
	window.addEventListener('resize', drawAllCanvases);

})


</script>
<link href="https://fonts.googleapis.com/css2?family=Barlow:wght@100..900&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://use.typekit.net/gkx8zdh.css">
<link rel="stylesheet" href="https://use.typekit.net/kha5kwj.css">
<link rel="stylesheet" href="https://use.typekit.net/mhl8zjo.css">
<link rel="stylesheet" href="https://s3.amazonaws.com/static.getsnworks.com/fontawesome/5.15.2/css/all.min.css">

<div class="container">
	<div class="content-container">
		<div class = "narrator-name" class:on={selectedTick}>
			{#if selectedTick}
				{narratorName} <span class="italic">interviewed by</span> {interviewerName}
			{/if}
		</div>
		<div class = "text-section-container">	
			<div class="narrator"></div>
			<div class="question"class:on={!selectedTick}>{hoveredTickQuestion}</div>
			<div class="answer" class:on={selectedTick}>{hoveredTickAnswer}</div>
			{#if !hoveredTickAnswer && !selectedTick}
				<div class="default">{@html defaultText}</div>
			{:else if !hoveredTickAnswer && selectedTick}
				<div class="default">{@html defaultFilterText}</div>
			{/if}
		</div>
		<div class="timeline">
		<div class = "row-container" bind:clientHeight={timelineHeight} bind:clientWidth={timelineWidth}>
			<div class="label">Pearl Harbor</div>
			<canvas id="pearl_harbor"></canvas>
		</div>
		<div class = "row-container">
			<div class=label>Belongings</div>

			<canvas id="belongings"></canvas>
			</div>
		<div class="row-container">
			<div class=label>Loyalty Questionnaire</div>
			<canvas id="questionnaire"></canvas>
		</div>
		<div class = "row-container">
			<div class=label>Hiroshima & Nagasaki</div>
			<canvas id="hiroshima"></canvas>
		</div>
		<div class = "row-container">
			<div class=label>Race</div>
			<canvas id="race"></canvas>
		</div>
		<div class = "row-container">
			<div class=label>Food</div>

			<canvas id="food"></canvas>
		</div>
		<div class="axis-label">Narrative Time</div>
		</div>
	</div>
</div>

<style>

	:global(a) {
		color: #fad0c6;
		text-decoration: underline;
		text-decoration-color: #fad0c6;
	}

	.container {
		background-color: rgb(252, 248, 244);
		background-color: white;
		background-color: #BA3F1D;
	}

	.content-container {
		display: flex;
		flex-direction: column;
		width: 80%;
		margin: auto;
		height: 100vh;
		justify-content: flex-end;
		padding-top: 0%;
		padding-bottom: 2%;
	}

	.narrator-name {
		width: 65%;
		margin: 1% auto 1%;
		height: 1.6rem;
		font-size: .8rem;
		text-align: center;
		vertical-align: baseline;
		color: #2a0c03d3;
		opacity: .1;
		font-weight: 600;
		transition: opacity .1s ease;
		font-family: "itc-avant-garde-gothic-pro";
	}

	.narrator-name.on {
		opacity: 1
	}

	.italic {
		font-weight: 500;
		font-style: italic;
		font-size: .9rem;
		font-family: "minion-pro", "itc-avant-garde-gothic-pro","franklin-gothic-urw","adobe-garamond-pro", "Georgia",  Georgia, Helvetica, sans-serif;

	}
	.text-section-container {
		height: 50%;
		width: 65%;
		margin: auto;
		display: flex;
		flex-direction: column;
		gap: 5%;
		overflow: scroll;
		margin-top: .0%;
		margin-bottom: 0%;

	}
	.question {
		text-align: center;
		font-size: 1.25rem;
		opacity: .8;
		font-family: "minion-pro", "itc-avant-garde-gothic-pro","franklin-gothic-urw","adobe-garamond-pro", "Georgia",  Georgia, Helvetica, sans-serif;
		color: #fad0c6;

	}

	.question.on {
		opacity: 1;
	}

	.answer {
		text-align: center;
		font-size: 1.2rem;
		opacity: .4;
		font-family: "Atlas Grotesk";
		color: #fad0c6;
		font-family: "minion-pro", "itc-avant-garde-gothic-pro","franklin-gothic-urw","adobe-garamond-pro", "Georgia",  Georgia, Helvetica, sans-serif;
	}

	.answer.on {
		opacity: 1;
		color: "#fad0c6"
	}

	.default {
		text-align: center;
		font-family: "minion-pro", "itc-avant-garde-gothic-pro","franklin-gothic-urw","adobe-garamond-pro", "Georgia",  Georgia, Helvetica, sans-serif;
		color: #fad0c6;
		font-size: 1rem;
		opacity: .8;
		width: 50%;
		margin: auto;
		margin-top: 8%;

		font-family: "itc-avant-garde-gothic-pro";

	}
	.timeline {
		display: grid;
		grid-template-rows: repeat(6, minmax(0, 1fr));
		width: 100%;
		height: 30.5svh;
		margin: 5% auto 0%;
	}

	.timeline .row-container {
		display: grid;
		grid-template-columns: 1fr minmax(0, 65%) 1fr;
		align-items: center;
		width: 100%;
		height: 100%;
		min-height: 0;
		min-width: 0;
		overflow: hidden;
	}

	.row-container .label {
		grid-column: 1;
		justify-self: end;
		width: 100%;
		text-align: right;
		padding-right: 12px;
		font-size: .9rem;
		opacity: 1;
		font-weight: 500;
		color: #fad0c6;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
		font-family: "itc-avant-garde-gothic-pro";
		/* font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif; */

	}
	.axis-label {
		/* margin-top: 4%; */
		margin: .75% auto 0;		
		font-size: .9rem;
		opacity: .5;
		color: #fad0c6;
		font-family: "itc-avant-garde-gothic-pro";

	}

	.timeline canvas {
		grid-column: 2;
		display: block;
		width: 100%;
		height: 100%;
		margin-left: 1%;
	}
</style>

