<script lang="ts">
    import PdfViewer from "$lib/PdfViewer.svelte";
    export const ssr = false; 
</script>
<div class="pdf-viewer">
    <PdfViewer url="./sample.pdf" />
</div>


<style>
    .pdf-viewer{
        width: 1200px;
        height: auto;
        aspect-ratio: 2/3;
    }
</style>