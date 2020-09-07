<script>
	import Editor from 'tailwind-editor'
	import HtmlFormatter from '../components/HtmlFormatter.svelte'
	let html = ""
	let arr_html = [
		{html:`Hello!`, klass:'p-2 text-4xl'},
		{html:`This is an example of a text description created by this 
			<span class="font-bold">editor.</span>
			<br>You can update and add section as you 
			<span class="underline">like,</span> or you can add 
			<span class="font-mono px-2 border border-gray-200 text-red-700">code</span>
			<span class="text-gray-800">&nbsp;and style it!</span>
			<br>
			<br>`, klass:'p-2'},
		{html:`&lt;h1&gt; code sections are not yet highlighted &lt;/h1&gt;`, klass:'p-2 code text-sm font-mono px-8 py-6 bg-gray-200'},
		{html:``, klass:'p-2'},
		{html:`<span class="font-bold">Our Roadmap</span>`, klass:'p-2 text-xl'},
		{html:`1. Add some list
  <br>2. Add embedded image and video
  <br>3. highlight code with just tailwind`, klass:'p-2 text-base'},
		{html:``, klass:'p-2'},
		{html:`This editor is built by 
  <a href="https://fouita.com" target="_blank" class="underline text-blue-500 link">fouita</a> team
`, klass:'p-2 text-base'}
	]

</script>

<svelte:head>
	<title>Tailwind Editor</title>
</svelte:head>

<div class="flex flex-row-reverse px-20 py-2">
	<a href="https://github.com/fouita/tailwind-editor" target="_blank">
		<img alt="github" src="https://img.icons8.com/fluent/32/000000/github.png"/>
	</a>
</div>
<div class="px-20 py-10 flex">
	<div class="flex-shrink-0 w-1/2"> 
		<Editor bind:html={html} {arr_html} />
	</div>
	<HtmlFormatter str={html} />
</div>