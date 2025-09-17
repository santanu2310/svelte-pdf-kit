<script lang="ts">
	import type { PDFPageProxy } from 'pdfjs-dist';

	let { pageProxy, page_no }: { pageProxy: PDFPageProxy; page_no: number } = $props();

	let canvas_parent: HTMLDivElement | null = $state(null);
	let canvas: HTMLCanvasElement;
	const dpr = window.devicePixelRatio || 1;
	let renderPage: (page: PDFPageProxy) => void; // Function to render single page

	$effect(() => {
		let parentWidth = canvas_parent!.clientWidth;
		let ctx = canvas.getContext('2d');

		renderPage = (page: PDFPageProxy) => {
			if (!ctx) {
				console.error('Could not get 2D context from canvas');
				return;
			}
			const unscaledViewport = page.getViewport({ scale: 1 });
			const scale = (parentWidth / unscaledViewport.width) * dpr;

			var viewport = page.getViewport({ scale: scale });

			// Canvas backing store size
			canvas.height = viewport.height;
			canvas.width = viewport.width;

			// CSS display size
			canvas.style.height = `${viewport.height / dpr}px`;
			canvas.style.width = `${viewport.width / dpr}px`;

			var renderContext = {
				canvasContext: ctx,
				canvas: canvas,
				viewport: viewport
			};
			page.render(renderContext);
		};

		renderPage(pageProxy);
	});
</script>

<div class="parent-div" bind:this={canvas_parent}>
	<span class="page-no">{page_no}</span>
	<canvas bind:this={canvas} id="the-canvas"></canvas>
</div>

<style>
	.parent-div {
		width: 100%;
		height: 100%;
		position: relative;
	}

	.page-no {
		width: fit-content;
		height: fit-content;
		padding: 2px 5px;
		background: rgba(2, 24, 41, 0.379);
		color: antiquewhite;
		position: absolute;
		left: 1px;
		bottom: 1px;
		border-top-right-radius: 6px;
	}

	#the-canvas {
		display: block;
		box-sizing: border-box;
		border: 1px solid #0419162f;
	}
</style>
