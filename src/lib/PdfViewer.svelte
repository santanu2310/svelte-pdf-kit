<script lang="ts">
	import { onMount } from 'svelte';
	import type { PDFDocumentProxy } from 'pdfjs-dist';
	let { url } = $props();

	let num = $state();
	let pdfDoc: PDFDocumentProxy | null = null;
	let pageNum = 1;
	let pageRendering = false;
	let pageNumPending: number | null = null;
	let scale = 0.8;
	let canvas_parent: HTMLDivElement | null = $state(null);

	let canvas: HTMLCanvasElement;
	let renderPage: Function;

	onMount(async () => {
		const pdfjsLib = await import('pdfjs-dist');

		pdfjsLib.GlobalWorkerOptions.workerSrc = './pdf.worker.min.mjs';

		let parentWidth = canvas_parent!.clientWidth;
		// let ctx: CanvasRenderingContext2D | null = canvas.getContext('2d');

		renderPage = (num: number) => {
      if (!pdfDoc) {
				return;
			}
			pageRendering = true;
			// Using promise to fetch the page
			pdfDoc.getPage(num).then(function (page) {
				const unscaledViewport = page.getViewport({ scale: 1 });
				const scale = parentWidth / unscaledViewport.width;
				var viewport = page.getViewport({ scale: scale });
				canvas.height = viewport.height;
				canvas.width = viewport.width;

				// Render PDF page into canvas context
				var renderContext = {
					canvas: canvas,
					viewport: viewport
				};
				var renderTask = page.render(renderContext);

				// Wait for rendering to finish
				renderTask.promise.then(function () {
					pageRendering = false;
					if (pageNumPending !== null) {
						// New page rendering is pending
						renderPage(pageNumPending);
						pageNumPending = null;
					}
				});
			});
		};

		pdfjsLib.getDocument(url).promise.then(function (pdfDoc_) {
			pdfDoc = pdfDoc_;
			renderPage(pageNum);
		});
	});

	function queueRenderPage(num: number) {
		if (pageRendering) {
			pageNumPending = num;
		} else {
			renderPage(num);
		}
	}

	function onPrevPage() {
		if (pageNum <= 1) {
			return;
		}
		pageNum--;
		queueRenderPage(pageNum);
	}

	function onNextPage() {
		if (pageNum >= pdfDoc!.numPages) {
			return;
		}
		pageNum++;
		queueRenderPage(pageNum);
	}
</script>

<div class="parent-div" bind:this={canvas_parent}>
	<div>
		<button id="prev" onclick={onPrevPage}>Previous</button>
		<button id="next" onclick={onNextPage}>Next</button>
		&nbsp; &nbsp;
		<span>Page: <span id="page_num">{num}</span> / <span id="page_count"></span></span>
	</div>

	<canvas width="" bind:this={canvas} id="the-canvas"></canvas>
</div>

<style>
	.parent-div {
		width: 100%;
		height: 100%;
	}
</style>
