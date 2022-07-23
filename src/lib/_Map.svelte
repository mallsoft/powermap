<script context="module" lang="ts">
	export type MapEntry = {
		name: 'NO1' | 'NO2' | 'NO3' | 'NO4' | 'NO5';
		from_UTC: string; // HH:MM
		to_UTC: string;
		price_NOK_KWh: number;
	};
</script>

<script lang="ts">
	import { d } from '$lib/_paths';
	export let entries: MapEntry[];

	$: entries.sort((a, b) => {
		if (a.price_NOK_KWh > b.price_NOK_KWh) return -1;
		if (a.price_NOK_KWh < b.price_NOK_KWh) return 1;
		return 0;
	});

	$: winner = entries.reduce((acc: MapEntry, cur: MapEntry) => {
		return acc.price_NOK_KWh > cur.price_NOK_KWh ? acc : cur;
	});

	$: looser = entries.reduce((acc: MapEntry, cur: MapEntry) => {
		return acc.price_NOK_KWh < cur.price_NOK_KWh ? acc : cur;
	});

	function round2dec(n: number): string {
		return (Math.round(n * 100) / 100).toFixed(2);
	}
</script>

{#if entries}
	<div class="powermap">
		<svg viewBox="0 0 600 750">
			<desc>Map of power prices in Norway in NOK per kWh.</desc>
			{#each entries as { name, price_NOK_KWh } (name)}
				<path d={d[name]}>
					<desc>{round2dec(price_NOK_KWh)} NOK/kWh - {name}</desc>
				</path>
			{/each}
		</svg>

		<ul>
			{#each entries as { name, price_NOK_KWh }}
				<li class={name} class:winner={winner.name === name}>
					<h2 class="name">{name}</h2>
					<p class="price">
						{round2dec(price_NOK_KWh)}
					</p>
				</li>
			{/each}
		</ul>

		<p>
			Å varme opp vannet for en 10 minutters dusj som bruker ca 10 liter i minuttet koster nå <em
				title="markeds prisen"
				>uten påslag
			</em>
			ca <strong>{round2dec(winner.price_NOK_KWh * 4)},-</strong>
			i sone {winner.name} eller <strong>{round2dec(looser.price_NOK_KWh * 4)},-</strong> i {looser.name}.
		</p>
	</div>
{/if}

<style>
	* {
		box-sizing: border-box;
	}

	div {
		--h1: #961938;
		--p: #ffffff;
		--li: #e44c4c;
		--path: #4e7cfc;

		position: relative;
		max-width: max-content;
	}

	svg {
		display: block;
		max-width: 600px;

		fill: var(--path);
		stroke: rgba(255, 255, 255, 0.5);
		stroke-width: 2;
		filter: drop-shadow(10px 10px 5px rgba(33, 87, 51, 0.1))
			drop-shadow(3px 3px 5px rgba(24, 66, 43, 0.2));
	}

	h2 {
		font-family: var(--hfont);
		font-size: 1em;
		margin: 0;
		margin-top: -0.25em;
		margin-bottom: 0.25em;
		text-align: center;
		color: var(--h1);
		font-weight: bold;
	}

	ul {
		height: 100%;
		width: 100%;
		top: 0;
		left: 0;
		position: absolute;

		list-style: none;
		padding: 0;
		margin: 0;

		pointer-events: none;
	}

	li {
		pointer-events: auto;

		position: absolute;

		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;

		background-color: var(--li);

		padding: 0.8em;
		font-size: 0.8em;
		font-weight: bold;
		border-radius: 100%;
		aspect-ratio: 1;

		border: rgba(0, 0, 0, 0.1) solid 2px;

		box-shadow: 2px 2px 7px -2px rgba(0, 0, 0, 0.7);
	}

	li > p {
		margin: 0;
		font-size: 1.6em;
		font-weight: bold;
		color: var(--p);

		text-shadow: 2px 2px 2px rgba(0, 0, 0, 0.8);
	}

	li.winner {
		border: rgba(0, 0, 0, 0.3) solid 3px;
	}

	li.winner::after {
		font-size: 2em;
		content: '🏆';
		position: absolute;
		top: -1em;
	}

	div > p {
		position: absolute;
		top: 35%;
		right: 3%;

		margin: 0;

		font-family: var(--hfont);

		padding: 8px;
		color: var(--h1);

		line-height: 1.4;
		word-spacing: 0.1em;

		max-width: 40%;
	}

	.NO1 {
		top: 72%;
		left: 32%;
		transform: rotate(10deg);
	}

	.NO2 {
		top: 90%;
		left: 14%;
		transform: rotate(-5deg);
	}

	.NO3 {
		top: 55%;
		left: 10%;
		transform: rotate(10deg);
	}

	.NO4 {
		top: 10%;
		left: 50%;
		transform: rotate(10deg);
	}

	.NO5 {
		top: 73%;
		left: 0%;
		transform: rotate(-10deg);
	}
</style>
