<svelte:options />

<script context="module" lang="ts">
	export type Zone = {
		name: 'NO1' | 'NO2' | 'NO3' | 'NO4' | 'NO5';
		data: {
			from_UTC: string; // HH:MM
			to_UTC: string;
			price_NOK_KWh: number;
		}[];
	};
</script>

<script lang="ts">
	import { dev } from '$app/env';

	import { onMount } from 'svelte';
	import Map from '$lib/_Map.svelte';
	import type { MapEntry } from '$lib/_Map.svelte';

	export let endpoint = 'https://powaaa.herokuapp.com/';

	if (!dev) {
		console.log(`
<Citations>
	https://fixer.io/ - API for currency conversion
	https://transparency.entsoe.eu/- Day ahead prices (EUR/MWh)
	https://mallsoft.dev/ - for more info
</Citations>
	`);
	}

	let time = new Date();
	$: HH_00 = `${time.getUTCHours()}:00`.padStart(5, '0');
	$: console.log(HH_00);

	async function getTreasonData(): Promise<Zone[]> {
		const response = await fetch(endpoint);
		const data = await response.json();

		return data;
	}

	function mapEntries(v: Zone[], t: string): MapEntry[] {
		const m = v.map((z) => {
			const found = z.data.find(({ from_UTC }) => from_UTC === t);
			return {
				name: z.name,
				...found
			} as MapEntry;
		});

		return m;
	}

	let promise = getTreasonData();

	onMount(() => {
		const interval1 = setInterval(() => {
			promise = getTreasonData();
		}, 1000 * 60 * 60);

		const interval2 = setInterval(() => {
			time = new Date();
		}, 1000);

		return () => {
			clearInterval(interval1);
			clearInterval(interval2);
		};
	});
</script>

<section>
	<h1>Nåværende kraftpris (nok/kWt)</h1>
	{#await promise}
		<p>...loading data ⚡..</p>
	{:then value}
		<Map entries={mapEntries(value, HH_00)} />
	{:catch err}
		<p>Sorry, {err}</p>
	{/await}
</section>

<style>
	section {
		font-family: 'Courier New', Courier, monospace;
		--hfont: 'Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode', Geneva,
			Verdana, sans-serif;

		padding: 8px;
		font-size: 10px;

		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
		gap: 20px;
	}

	@media (min-width: 570px) {
		section {
			font-size: 16px;
		}
	}

	h1 {
		order: 2;
		max-width: 35ch;
		font-size: 1em;
		font-style: italic;
		color: rgb(92, 114, 107);
		margin: 0;
		padding: 0;
	}
</style>
