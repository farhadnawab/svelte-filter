<script>
	import RangeSlider from './RangeSlider.svelte';
	import { onMount } from "svelte";
	import { writable, derived } from 'svelte/store';

	let start;
	let end;
	let groupNumber = 7;
	let maxPrice = 0;
	const nice = d => {
		if (!d && d !== 0) return '';
		return d.toFixed(2);
	}

	function dataFactory(data, numberOfBins) {
		var max = Math.max.apply(Math, data.map(function(o) { return o.price; }));
		max = Math.floor(max);
		var groupSize = Math.ceil(max/numberOfBins);
		var result = new Array(groupSize).fill(0);

		maxPrice = max;

		for (var i = 0; i < data.length; i++) {
			var index = data[i].price/numberOfBins;
			index = Math.floor(index);
			result[index] = result[index] == undefined ? 1 : result[index]+1;
		}

		return result;
	}

	let histogramData = writable([]);

	onMount(async () => {
		fetch("https://development-dot-petfoodcompare.wl.r.appspot.com/v1/pet/products-list?species=dog")
		.then(response => response.json())
		.then(res => {
			histogramData.set(dataFactory(res, groupNumber));

				console.log("data", $histogramData);
		}).catch(error => {
			console.log(error);
			return [];
		});

	});

</script>

<main>
	<div class="container">
		<div class="row">
			<div class="col-12 col-md-3">
				{#if maxPrice > 0}
				<RangeSlider bind:start bind:end bind:histogramData bind:groupNumber/>
				<div class="slider-labels">
					<div class="label">${nice(start*maxPrice)}</div>
					<div class="label">${nice(end*maxPrice)}</div>
				</div>
				{:else}
					<center>Loading Filter...</center>
				{/if}
			</div>

			<div class="col-12 col-md-9">
				<h1>MultiRangeSlider</h1>

				<p>Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn how to build Svelte apps.</p>
			</div>
		</div>
	</div>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

	* {box-sizing:border-box; -webkit-box-sizing:border-box;}
	.container {max-width:1200px; padding:0; margin:0 auto; width:100%;}
	.row {margin-left:-15px; margin-right:-15px; display:flex; flex-wrap:wrap;}
	.col-md-3 {width:25%; flex:0 0 25%; padding-left:15px; padding-right:15px;}
	.col-md-9 {width:75%; flex:0 0 75%; padding-left:15px; padding-right:15px;}

	.slider-labels {display:flex; justify-content: space-between;font-size:12px;padding:5px 0;}
</style>