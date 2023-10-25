<script lang="ts">
	import { Heading, Button, Hr, P, Span, ButtonGroup, List, Li } from 'flowbite-svelte';
	import { Search } from 'flowbite-svelte';
	import Intro from '$lib/Intro.svelte';

	export let data;

	let pageInfo = '';
	let cruxMetrics = {};
	let lighthouseMetrics = {};
	let inputUrl = '';
	let searchStatus = 'Search';

	var a = 1;

	async function run() {
		const url = setUpQuery(inputUrl);
		searchStatus = 'Searching...';
		try {
			const response = await fetch(url);
			if (!response.ok) {
				throw new Error(`API request failed with status ${response.status}`);
			}
			const json = await response.json();
			showInitialContent(json.id);
			cruxMetrics = {
				'First Contentful Paint': json.loadingExperience.metrics.FIRST_CONTENTFUL_PAINT_MS.category,
				'First Input Delay': json.loadingExperience.metrics.FIRST_INPUT_DELAY_MS.category
			};

			lighthouseMetrics = {
				'First Contentful Paint':
					json.lighthouseResult.audits['first-contentful-paint']?.displayValue || 'N/A',
				'Speed Index': json.lighthouseResult.audits['speed-index']?.displayValue || 'N/A',
				'Time To Interactive': json.lighthouseResult.audits['interactive']?.displayValue || 'N/A',
				'First Meaningful Paint':
					json.lighthouseResult.audits['first-meaningful-paint']?.displayValue || 'N/A',
				'First CPU Idle': json.lighthouseResult.audits['first-cpu-idle']?.displayValue || 'N/A',
				'Estimated Input Latency':
					json.lighthouseResult.audits['estimated-input-latency']?.displayValue || 'N/A'
			};
			searchStatus = 'Searched';
			if (a == 1) {
				document.querySelector('#inTheend').style.opacity = '1';
				document.querySelector('#inTheend').style.scale = '1';
				return (a = 0);
			}
		} catch (error) {
			console.log('Error:', error);
		}
	}

	function setUpQuery(url) {
		if (!url.startsWith('http://') && !url.startsWith('https://')) {
			url = 'http://' + url;
		}
		const api = 'https://www.googleapis.com/pagespeedonline/v5/runPagespeed';
		const parameters = {
			url: encodeURIComponent(url)
		};
		// let queryString = Object.keys(parameters).map((key) => `${key}=${encodeURIComponent(parameters[key])}`
		let query = `${api}?`;
		for (const key in parameters) {
			query += `${key}=${parameters[key]}&`;
		}
		return query;
	}

	function showInitialContent(id) {
		pageInfo = `${id}`;
	}

	var btn = document.querySelector('.btn');

	function reset() {
		pageInfo = '';
		cruxMetrics = {};
		lighthouseMetrics = {};
		inputUrl = '';
		searchStatus = 'Search';
		document.querySelector('#inTheend').style.opacity = '0';
		document.querySelector('#inTheend').style.scale = '0';
	}
</script>

<Intro heading={data.meta.title} description={data.meta.description} />

<section class="bg-white dark:bg-gray-900">
	<div class="insert">
		<!-- <label for="urlInput" class="tag">Enter URL:</label> -->
		<Search type="url" id="urlInput" bind:value={inputUrl} placeholder="Enter your URL">
			<ButtonGroup>
				<Button outline color="dark" on:click={run}>{searchStatus}</Button>
				<Button outline color="dark" on:click={reset}>Reset</Button>
			</ButtonGroup>
		</Search>
	</div>
	<div id="inTheend">
		<P color="text-green-700 dark:text-green-500"
			><Span class="uppercase">Website:</Span> {pageInfo}
		</P>
		<Hr classHr="my-8" />
		<Heading tag="h4">User Experience Report Results</Heading>
		<P whitespace="preline">
			{#each Object.keys(cruxMetrics) as key}
				<li class="out">{key}: {cruxMetrics[key]}</li>
			{/each}
		</P>
		<Hr classHr="my-8" />
		<Heading tag="h4">Lighthouse Results</Heading>
		<P whitespace="preline">
			{#each Object.keys(lighthouseMetrics) as key}
				<li class="out">{key}: {lighthouseMetrics[key]}</li>
			{/each}
		</P>
	</div>
</section>

<style>
	section {
		width: 85%;
		margin-left: auto;
		margin-right: auto;
		border: 2px solid gray;
		border-radius: 10px;
		padding: 10px;
	}
	#inTheend {
		border: 2px solid rgb(129, 129, 129);
		width: 100%;
		padding: 10px;
		display: block;
		align-items: center;
		margin: left;
		margin-top: 10px;
		opacity: 0;
		scale: 0;
	}
	.out {
		list-style: none;
		color: #000;
		background-color: beige;
		align-items: left;
		justify-content: left;
		padding: 05px;
	}
</style>
