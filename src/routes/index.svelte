<script>
	import Editor from 'tailwind-editor'
	import HtmlFormatter from '../components/HtmlFormatter.svelte'
	let html = ""


</script>

<svelte:head>
	<title>Tailwind Editor</title>
</svelte:head>

<div class="px-20 py-10 flex">
	<div class="flex-shrink-0 w-1/2"> 
		<Editor bind:html={html} />
	</div>
	<HtmlFormatter str={html} />
</div>