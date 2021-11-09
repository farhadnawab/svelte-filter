<script>
	import { clamp } from 'yootils';
	import { onMount } from "svelte";
	import { writable, derived } from 'svelte/store';


	export let start = 0;
	export let end = 1;
	export let histogramData = null;
	export let groupNumber = null;
	let leftHandle;
	let body;
	let slider;
	
	function draggable(node) {
		let x;
		let y;

		function handleMousedown(event) {
			if (event.type === 'touchstart') {
				event = event.touches[0];
			}
			x = event.clientX;
			y = event.clientY;
			node.dispatchEvent(new CustomEvent('dragstart', {
				detail: { x, y }
			}));
			window.addEventListener('mousemove', handleMousemove);
			window.addEventListener('mouseup', handleMouseup);
			window.addEventListener('touchmove', handleMousemove);
			window.addEventListener('touchend', handleMouseup);
		}
		
		function handleMousemove(event) {
			if (event.type === 'touchmove') {
				event = event.changedTouches[0];
			}
			const dx = event.clientX - x;
			const dy = event.clientY - y;
			x = event.clientX;
			y = event.clientY;
			node.dispatchEvent(new CustomEvent('dragmove', {
				detail: { x, y, dx, dy }
			}));
		}
		
		function handleMouseup(event) {
			x = event.clientX;
			y = event.clientY;
			node.dispatchEvent(new CustomEvent('dragend', {
				detail: { x, y }
			}));
			window.removeEventListener('mousemove', handleMousemove);
			window.removeEventListener('mouseup', handleMouseup);
			window.removeEventListener('touchmove', handleMousemove);
			window.removeEventListener('touchend', handleMouseup);
		}
		
		node.addEventListener('mousedown', handleMousedown);
		node.addEventListener('touchstart', handleMousedown);
		
		return {
			destroy() {
				node.removeEventListener('mousedown', handleMousedown);
				node.removeEventListener('touchstart', handleMousedown);
			}
		};
	}
	
	function setHandlePosition (which) {
		return function (evt) {
			const { left, right } = slider.getBoundingClientRect();
			const parentWidth = right - left;
			const p = Math.min(Math.max((evt.detail.x - left) / parentWidth, 0), 1);
			if (which === 'start') {
				start = p;
				end = Math.max(end, p);
			} else {
				start = Math.min(p, start);
				end = p;
			}
		}
	}
	function setHandlesFromBody (evt) {
		const { width } = body.getBoundingClientRect();
		const { left, right } = slider.getBoundingClientRect();
		const parentWidth = right - left;
		const leftHandleLeft = leftHandle.getBoundingClientRect().left;
		const pxStart = clamp((leftHandleLeft + event.detail.dx) - left, 0, parentWidth - width);
		const pxEnd = clamp(pxStart + width, width, parentWidth);
		const pStart = pxStart / parentWidth;
		const pEnd = pxEnd / parentWidth;
		start = pStart;
		end = pEnd;
	}

	// function dataFactory(data, numberOfBins) {
	// 	var max = Math.max.apply(Math, data.map(function(o) { return o.price; }));
	// 	max = Math.floor(max);
	// 	var groupSize = Math.ceil(max/numberOfBins);
	// 	var result = new Array(groupSize).fill(0);

	// 	end = max;

	// 	for (var i = 0; i < data.length; i++) {
	// 		var index = data[i].price/numberOfBins;
	// 		index = Math.floor(index);
	// 		result[index] = result[index] == undefined ? 1 : result[index]+1;
	// 	}

	// 	return result;
	// }

	// const apiData = writable([]);
	// const groupNumber = 10;

	// onMount(async () => {
	// 	fetch("https://development-dot-petfoodcompare.wl.r.appspot.com/v1/pet/products-list?species=dog")
	// 	.then(response => response.json())
	// 	.then(res => {
	// 		apiData.set(dataFactory(res, groupNumber));

	// 			console.log("data", $apiData);
	// 	}).catch(error => {
	// 		console.log(error);
	// 		return [];
	// 	});

	// });

</script>

<div class="double-range-container">
	<div class="histogram">
		{#each $histogramData as item, i}
			<div style="height: {item === 0 ? 4 : ((item/15)+4)}px" count="{item}">
				<span class="tooltip">
					{item} <br>
					${(i*groupNumber)+1} - ${((i+1)*groupNumber)+1}
				</span>
			</div>
		{/each}
	</div>
	<div class="slider" bind:this={slider}>
		<div
			class="body"
			bind:this={body}
			use:draggable
			on:dragmove|preventDefault|stopPropagation="{setHandlesFromBody}"
			style="
				left: {100 * start}%;
				right: {100 * (1 - end)}%;
			"
			></div>
		<div
			class="handle"
			bind:this={leftHandle}
			data-which="start"
			use:draggable
			on:dragmove|preventDefault|stopPropagation="{setHandlePosition('start')}"
			style="
				left: {100 * start}%
			"
		></div>
		<div
			class="handle"
			data-which="end"
			use:draggable
			on:dragmove|preventDefault|stopPropagation="{setHandlePosition('end')}"
			style="
				left: {100 * end}%
			"
		></div>
	</div>
</div>

<style>
	.double-range-container {
		width: 100%;
		user-select: none;
		box-sizing: border-box;
		white-space: nowrap
	}
	.slider {
		position: relative;
		width: 100%;
		height: 3px;
		top: 50%;
		transform: translate(0, -50%);
		background-color: #e0e0e0;
		border-radius: 1px;
	}
	.handle {
		position: absolute;
		top: 50%;
		width: 0;
		height: 0;
	}
	.handle:after {
		content: ' ';
		box-sizing: border-box;
		position: absolute;
		border-radius: 50%;
		width: 16px;
		height: 16px;
		background-color: #4ba4fe;
		border: 3px solid #fff;
		box-shadow:1px 1px 2px 2px rgba(0,0,0,0.10);
		transform: translate(-50%, -50%)
	}
	/* .handle[data-which="end"]:after{
		transform: translate(-100%, -50%);
	} */
	.handle:active:after {
		background-color: #ddd;
		z-index: 9;
	}
	.body {
		top: 0;
		position: absolute;
		background-color: #ff6a00;
		bottom: 0;
	}

	.histogram {display:flex; align-items: end; position:relative; margin:0 0 4px;}
	.histogram .tooltip{position:absolute;top:0; display:none; background:#fff; border-radius:3px; font-size:11px; padding:5px; box-shadow:1px 1px 1px 2px rgba(0,0,0,0.2);}
	.histogram > div {width:100%; background:#eee; border-radius:3px 3px 0 0; margin:0 1px;}
	.histogram > div:hover .tooltip{display:block;}
</style>