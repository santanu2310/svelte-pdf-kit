<script lang="ts">
	import { onMount } from 'svelte';
	import type { PDFDocumentProxy, PDFPageProxy } from 'pdfjs-dist';
	import PdfPage from './components/PdfPage.svelte';

	interface PageLayout {
		pageNumber: number;
		pageProxy: PDFPageProxy;
		height: number;
		offsetTop: number;
	}

	let { url }: { url: string } = $props();

	let pdfDoc: PDFDocumentProxy | null = null;
	let allPages: PageLayout[] = $state([]);

	let isLoading = $state(true);
	let error: string | null = $state(null);

	let scrollTop = $state(0);
	let viewportHeight = $state(0);
	let viewportWidth = $state(0);

	const OVERSCAN = 2;

	// total height of pages combined
	const totalHeight = $derived(
		allPages.length > 0
			? allPages[allPages.length - 1].offsetTop + allPages[allPages.length - 1].height
			: 0
	);

	const startIndex = $derived(
		Math.max(0, allPages.findIndex((p) => p.offsetTop + p.height >= scrollTop) - OVERSCAN)
	);

	const endIndex = $derived(() => {
		const foundIndex = allPages.findIndex((p) => p.offsetTop >= scrollTop + viewportHeight);

		const boundaryIndex = foundIndex === -1 ? allPages.length : foundIndex;
		return Math.min(allPages.length, boundaryIndex + OVERSCAN);
	});

	const visiblePages = $derived(allPages.slice(startIndex, endIndex()));

	onMount(async () => {
		try {
			const pdfjsLib = await import('pdfjs-dist');

			pdfjsLib.GlobalWorkerOptions.workerSrc = new URL(
				'pdfjs-dist/build/pdf.worker.mjs',
				import.meta.url
			).toString();

			pdfDoc = await pdfjsLib.getDocument(url).promise;

			if (!pdfDoc) throw new Error('Failed to load pdf.');
			const loadedDoc = pdfDoc;
			const numPages = loadedDoc.numPages;

			const pagePromises = Array.from({ length: numPages }, (_, i) => loadedDoc.getPage(i + 1));
			const pageProxies = await Promise.all(pagePromises);

			const pageLayouts: PageLayout[] = [];
			let currentOffset = 0; // Space between pages to be added

			for (let i = 0; i < pageProxies.length; i++) {
				const proxy = pageProxies[i];
				const unscaledViewport = proxy.getViewport({ scale: 1 });

				const scale = viewportWidth / unscaledViewport.width;
				const scaledHeight = unscaledViewport.height * scale;

				pageLayouts.push({
					pageNumber: i + 1,
					pageProxy: proxy,
					height: scaledHeight,
					offsetTop: currentOffset
				});

				currentOffset += scaledHeight + 16;
			}

			allPages = pageLayouts;
		} catch (e) {
			console.error('Failed to load and process PDF:', e);
			error = 'Could not load the PDF file.';
		} finally {
			isLoading = false;
		}
	});
</script>

<div
	class="viewport"
	bind:clientHeight={viewportHeight}
	bind:clientWidth={viewportWidth}
	onscroll={(e) => (scrollTop = e.currentTarget.scrollTop)}
>
	{#if isLoading}
		<p>Loading document...</p>
	{:else if error}
		<p>{error}</p>
	{:else}
		<div class="sizer" style="height: {totalHeight}px;">
			{#each visiblePages as page (page.pageNumber)}
				<div class="page-wrapper" style="position: absolute; top: {page.offsetTop}px; width: 100%;">
					<PdfPage pageProxy={page.pageProxy} page_no={page.pageNumber} />
				</div>
			{/each}
		</div>
	{/if}
</div>

<style>
	.viewport {
		width: 100%;
		height: 100%;
		overflow-y: auto;
		background-color: #f0f0f0;
		box-sizing: border-box;
		text-align: center;
	}

	.viewport > p {
		margin-top: calc(50% - 9px);
		font-size: 18px;
	}
	.sizer {
		position: relative;
		width: 100%;
	}
	.page-wrapper {
		display: flex;
		justify-content: center;
	}
</style>
