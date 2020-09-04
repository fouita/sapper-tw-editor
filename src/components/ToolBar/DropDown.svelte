<script>
	// import {createEventDispatcher} from 'svelte/internal'
	import List from './List.svelte'
	import {fly} from 'svelte/transition'
	let klass = ''
	export {klass as class}
	export let selected
	// let dispatch = createEventDispatcher()

	export let list = [
		{
			label:'none',
			value:'none'
		},
		{
			label:'tight',
			value:'tight'
		},
		{
			label:'snug',
			value:'snug'
		},
		{
			label:'normal',
			value:'normal'
		},
		{
			label:'relaxed',
			value:'relaxed'
		},
		{
			label:'loose',
			value:'loose'
		}
	]

	let show_list = false
	function toggleList(e){
		show_list= !show_list
		e.preventDefault()
		e.stopPropagation()
	}
	
	function hideList(){
		show_list = false
	}
	
</script>

<svelte:window on:click={hideList} />
<div class="relative">
	<div class="cursor-pointer flex h-full" on:click={toggleList}>
		<slot>Click To show</slot>
	</div>
	{#if show_list}
	<div transition:fly class="absolute -ml-1 z-20 {klass}">
		<List {list} {selected} on:select />
	</div>
	{/if}
</div>