
<script>
	import ContentEditor from './ContentEditor.svelte'
	import ToolBar from '../ToolBar/ToolBar.svelte'
	import { tick } from 'svelte/internal';		
	
	let arr_html = [
		{html:`Hello!`, klass:'p-2'},
		{
			html: `sdklfjh skdjfh sdfs<br>sdfls kdfjsldk jsldf jlskdjsldkfl skdjfl ksdflksjdf lskdjfl skdfj slkjzelrkzerlzkjerlzkerlzkej rlzk jlze rlzkejr lkzelrkj<br>sdfksjdfh sdf<br>sdfkjsdfhsdf<br>dskfjshdfkjsdf<br>zerlzkerjzelr`,
			klass: 'p-2'
		}
	]
	
	$: gen_code = arr_html.map(h => `<div class='${h.klass}'>${h.html}</div>`).join('\n')
	async function addNewElm(i){
		
		arr_html.splice(i+1,0,{html: '',klass:'p-2'})
		// auto focus
		arr_html = arr_html
		await tick()
		list_editors.children[i+1].focus()
		
	}
	
	let list_editors
	function setListEditors(node){
		list_editors = node
	}
	
	let show_toolbar = false
	
	let setClass
	let setGClass
	let base_node
	let classes
	let g_classes
	let href

	function showToolBar(evt){
		base_node =  evt.detail.base_node
		if(!base_node || base_node.tagName == 'DIV') 
			return
		show_toolbar =  true
		setClass =  evt.detail.setClass
		setGClass =  evt.detail.setGClass
		classes =  evt.detail.classes
		g_classes =  evt.detail.g_classes
		href =  evt.detail.href
	}
	
	function hideSelect(){
		show_toolbar=false
	}
	
</script>

{#if show_toolbar}
	<ToolBar {setGClass} {setClass} {base_node} {g_classes} {classes} {href} on:close={hideSelect} />
{/if}

<div class="flex">
	<div use:setListEditors class="w-2/3 flex-shrink-0">
		{#each arr_html as h,i}
			<ContentEditor bind:html={h.html} bind:gklass={h.klass} on:enter={() => addNewElm(i)} on:select={showToolBar} on:hideselect={hideSelect} />
		{/each}
	</div>
	<div class="break-all">
		<pre class="whitespace-pre-line" >
			<code >{gen_code}</code>
		</pre>
	</div>
</div>

					  