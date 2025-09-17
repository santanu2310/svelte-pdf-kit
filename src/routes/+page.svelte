<script lang="ts">
	import PdfViewer from '$lib/PdfViewer.svelte';
	export const ssr = false;
</script>

<div class="page">
	<div class="pdf-viewer">
		<PdfViewer url="./sample.pdf" />
	</div>
</div>

<style>
	.page {
		width: 100svw;
		height: 100svh;
		display: flex;

		align-items: center;
		justify-content: center;
	}
	.pdf-viewer {
		width: 800px;
		height: 96%;
	}
</style>
