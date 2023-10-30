<script lang="ts">
	import {
		Heading,
		Button,
		Hr,
		P,
		Span,
		ButtonGroup,
		Gallery,
		List,
		Li,
		Popover
	} from 'flowbite-svelte';
	import { Search } from 'flowbite-svelte';
	import Intro from '$lib/Intro.svelte';

	export let data;

	let pageInfo = '';
	let cruxMetrics = {};
	let lighthouseMetrics = {};
	let inputUrl = '';
	let searchStatus = 'Search';

	function formatURL(url) {
		if (!url.startsWith('http://') && !url.startsWith('https://')) {
			// If the URL does not start with "http" or "https," add "http://" as the default prefix.
			url = 'http://' + url;
		}
		return url;
	}
	async function displayScreenshot(screenshotUrl: URL | RequestInfo) {
		const screenshot = document.getElementById('screenshot');
		const response = await fetch(screenshotUrl);
		const blob = await response.blob();
		screenshot.src = URL.createObjectURL(blob);
		// screenshot.style.display = 'block';
	}
	async function run() {
		const urlInput = document.querySelector('#urlInput'); // Correct the selector
		const url = formatURL(urlInput.value);
		searchStatus = 'Searching...';

		try {
			const response = await fetch(
				`https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=${url}`
			);
			if (!response.ok) {
				throw new Error(`API request failed with status ${response.status}`);
			}

			const results = await response.json();
			const websiteName = results.lighthouseResult.finalUrl || 'Website Name Not Available';
			showInitialContent(websiteName);

			// Display the speed test results in the results div.
			const speedTestResults = document.querySelector('#OPP');
			speedTestResults.innerHTML = ''; // Clear existing results

			// Convert JSON to HTML and append to the results div.
			const htmlResults = convertJSONToHTML(results.lighthouseResult.audits);
			htmlResults.forEach((item) => {
				speedTestResults.appendChild(item);
			});
			const overallScoreElement = document.getElementById('overall-score');
			const performanceScore = results.lighthouseResult.categories.performance.score * 100;
			overallScoreElement.textContent = performanceScore.toFixed(2) + '%';

			// Apply color based on the percentage
			if (performanceScore >= 90) {
				overallScoreElement.style.color = 'green';
			} else if (performanceScore >= 50) {
				overallScoreElement.style.color = 'yellow';
			} else {
				overallScoreElement.style.color = 'red';
			}

			searchStatus = 'Searched';
			document.querySelector('#inTheend').style.display = 'block';
			document.querySelector('#inTheend').style.opacity = '1';
			document.querySelector('#inTheend').style.scale = '1';
		} catch (error) {
			console.log('Error:', error);
		}
		const screenshotUrl = `https://api.microlink.io/?url=${url}&screenshot=true&embed=screenshot.url`;
		await displayScreenshot(screenshotUrl);
	}

	function convertJSONToHTML(jsonResults) {
		const htmlResults = [];

		for (const key in jsonResults) {
			const result = jsonResults[key];

			const cardElement = document.createElement('div');
			cardElement.classList.add('result-card');

			const h3Element = document.createElement('h3');
			h3Element.textContent = key;

			const pElement = document.createElement('p');
			pElement.textContent = `Score: ${result.numericValue}`;

			// Add color class based on the score value
			if (result.numericValue >= 90) {
				pElement.classList.add('score-good');
			} else if (result.numericValue >= 50) {
				pElement.classList.add('score-moderate');
			} else {
				pElement.classList.add('score-poor');
			}

			cardElement.appendChild(h3Element);
			cardElement.appendChild(pElement);

			htmlResults.push(cardElement);
		}

		return htmlResults;
	}
	function showInitialContent(websiteName) {
		pageInfo = websiteName;
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
		document.querySelector('#inTheend').style.display = 'none';
	}
</script>

<Intro heading={data.meta.title} description={data.meta.description} />

<section class="bg-white dark:bg-gray-900">
	<div class="insert">
		<!-- <label for="urlInput" class="tag">Enter URL:</label> -->
		<Search type="url" id="urlInput" bind:value={inputUrl} placeholder="Enter your URL">
			<Popover class="w-64 text-sm font-light " title="Search" triggeredBy="#urlInput"
				>Enter your website's name; you may use HTTP/HTTPS or search by name with correct domain
				name</Popover
			>
			<ButtonGroup>
				<Button id="b1" outline color="dark" on:click={run}>{searchStatus}</Button>
				<Popover class="w-64 text-sm font-light " title="Press to Search" triggeredBy="#b1"
					>You may also search the insides of your website from this page</Popover
				>
				<Button id="b2" outline color="dark" on:click={reset}>Reset</Button>
				<Popover class="w-64 text-sm font-light " title="Reset it!" triggeredBy="#b2"
					>And here's an option to reload the page</Popover
				>
			</ButtonGroup>
		</Search>
	</div>
	<div id="inTheend">
		<P color="text-green-700 dark:text-green-500"
			><Span class="uppercase center">Website:</Span>
			{pageInfo}
			<div class="ip" id="overall-performance" style="display: flex;">
				<Span class="uppercase center">Overall-Performance: &nbsp;</Span>
				<p><span id="overall-score" /> </p>
			</div>
		</P>
		<Gallery class="gap-5 grid">
			<img
				id="screenshot"
				alt="shoas"
				class="h-auto max-w-full rounded-lg ring-4 ring-red-400 dark:ring-red-400 p-1"
			/>
		</Gallery>
		<Hr classHr="my-3" />
		<Heading tag="h4">User Experience Report Results</Heading>
		<Hr classHr="my-1" />
		<Gallery id="OPP" class="gap-4 grid-cols-2 md:grid-cols-4" color-gray />
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
		color: rgb(105, 105, 105);
		padding: 10px;
		display: none;
		text-align: left;
		margin: center;
		margin-top: 10px;
		opacity: 0;
		scale: 0;
	}
	#screenshot {
		margin-left: auto;
		height: 60vh;
		width: 50vw;
	}
</style>
