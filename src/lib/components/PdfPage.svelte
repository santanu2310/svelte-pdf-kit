<script lang="ts">
	import type { PDFPageProxy } from 'pdfjs-dist';

	let { pageProxy, page_no }: { pageProxy: PDFPageProxy; page_no: number } = $props();

	let canvas_parent: HTMLDivElement | null = $state(null);

	let canvas: HTMLCanvasElement;
	let renderPage: (page: PDFPageProxy) => Promise<void>;

	$effect(() => {
		// pdfjsLib.GlobalWorkerOptions.workerSrc = './pdf.worker.min.mjs';

		let parentWidth = canvas_parent!.clientWidth;
		let ctx = canvas.getContext('2d');
		// let ctx: CanvasRenderingContext2D | null = canvas.getContext('2d');

		renderPage = async (page: PDFPageProxy) => {
			if (!ctx) {
				console.error('Could not get 2D context from canvas');
				return;
			}
			const unscaledViewport = page.getViewport({ scale: 1 });
			const scale = parentWidth / unscaledViewport.width;
			var viewport = page.getViewport({ scale: scale });
			canvas.height = viewport.height;
			canvas.width = viewport.width;

			// Render PDF page into canvas context

			var renderContext = {
				canvasContext: ctx,
				canvas: canvas,
				viewport: viewport
			};
			var renderTask = page.render(renderContext);

			// Wait for rendering to finish
			// renderTask.promise.then(function () {
			// 	pageRendering = false;
			// 	if (pageNumPending !== null) {
			// 		// New page rendering is pending
			// 		renderPage(pageNumPending);
			// 		pageNumPending = null;
			// 	}
			// });
		};

		renderPage(pageProxy);
	});

	// function queueRenderPage(num: number) {
	// 	if (pageRendering) {
	// 		pageNumPending = num;
	// 	} else {
	// 		renderPage(num);
	// 	}
	// }
</script>

<div class="parent-div" bind:this={canvas_parent}>
	<canvas width="" bind:this={canvas} id="the-canvas"></canvas>
</div>

<style>
	.parent-div {
		width: 100%;
		height: 100%;
	}
</style>
