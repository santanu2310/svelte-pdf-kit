<script lang="ts">
	import { onMount } from 'svelte';
	import type { PDFDocumentProxy, PDFPageProxy } from 'pdfjs-dist';
	import PdfPage from './components/PdfPage.svelte';

	interface PageData {
		pageNumber: number;
		pageProxy: PDFPageProxy;
	}

	let { url }: { url: string } = $props();

	let pdfDoc: PDFDocumentProxy | null = null;
	let scale = 0.8;
	let canvas_parent: HTMLDivElement | null = $state(null);

	let pages: PageData[] = $state([]);
	let isLoading = $state(true);
	let error: string | null = $state(null);

	onMount(async () => {
		try {
			const pdfjsLib = await import('pdfjs-dist');

			pdfjsLib.GlobalWorkerOptions.workerSrc = './pdf.worker.min.mjs';

			pdfDoc = await pdfjsLib.getDocument(url).promise;

			if (!pdfDoc) throw new Error();
			const loadedDoc = pdfDoc;
			const numPages = loadedDoc.numPages;

			const pagePromises = Array.from({ length: numPages }, (_, i) => loadedDoc.getPage(i + 1));
			const pageProxies = await Promise.all(pagePromises);

			pages = pageProxies.map((proxy, index) => ({
				pageNumber: index + 1,
				pageProxy: proxy
			}));
		} catch (e) {
			console.error('Failed to load and process PDF:', e);
			error = 'Could not load the PDF file.';
		} finally {
			isLoading = false;
		}

	});
</script>

<div class="parent-div" bind:this={canvas_parent}>
	{#if isLoading}
		<p>Loding document...</p>
	{:else if error}
		<p>Error</p>
	{:else}
		{#each pages as page (page.pageNumber)}
			<div class="page-wrapper">
				<PdfPage pageProxy={page.pageProxy} page_no={page.pageNumber} />
			</div>
		{/each}
	{/if}
</div>

<style>
	.parent-div {
		width: 100%;
		height: 100%;
	}
</style>
