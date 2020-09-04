
<script>
	import { tick, createEventDispatcher, onMount } from 'svelte/internal';
	let dispatch = createEventDispatcher()

	let arr_elms = [{"txt":"The follo"},{"txt":"The \nfollo"},{"txt":"win","klass":"font-bold"},{"txt":" ok"},{"tag":"BR",txt: ""},{"txt":"155\ng will return true if nothing is selected."}]
	
	export let html = ''
	export let gklass = ''
	
	function generateArr(){
		let div = document.createElement('div')
		div.innerHTML = html
		
		let n_elms = []
		for(let elm of [...div.childNodes]){
			
			if(elm.nodeName == 'BR')
				n_elms.push({tag: 'BR',txt: ""})
			else if(elm.nodeName == 'A')
				n_elms.push({tag: 'A',txt: elm.textContent, href: elm.getAttribute('href'), klass: elm.classList&&[...elm.classList].join(' ')})
			else if(elm.nodeName !== '#text')
				n_elms.push({txt: elm.innerText, klass: elm.classList&&[...elm.classList].join(' ')})
			else if(elm.nodeName == '#text' && elm.length>0 ){
				n_elms.push({txt: elm.textContent})
				
			}
		}
		
		arr_elms = n_elms
	}
	let mounted = false
	$: if(html && mounted){
		// generate arr_elms
		generateArr()
	}

	onMount(() => {
		mounted = true
		generateArr()
	})
	
	async function handleKeydown(e){

		// console.log(e.keyCode)
		if([38,40,8].includes(e.keyCode)){

			let selection = window.getSelection()
			// up key
			let b_node = selection.baseNode
			let e_node = selection.extentNode
			let start_i = selection.baseOffset
			let end_i = selection.extentOffset
			let elm_node = (b_node.tagName&&b_node.tagName=='DIV') ? b_node : b_node.parentNode.tagName == 'DIV' ? b_node.parentNode : b_node.parentNode.parentNode
			
			let b_index = getIndex(b_node)
			let e_index = getIndex(e_node)

			if(e.keyCode == 38){ 
				// console.log(e.keyCode," start_i ", start_i, ' b_node ',b_node , ' e_node ',e_node)
				console.log('elm node ... ', elm_node)
				console.log('B index --> ', b_index)
				if (b_node == e_node && start_i == 0 && (b_index <= 1)){
					// move to the previous node
					let pv_elm = elm_node.previousElementSibling
					console.log("PV ELM -------------- ", pv_elm)
					if(pv_elm && pv_elm.isContentEditable){
						pv_elm.focus()
						console.log("FOCUS *---> PV ELM ", pv_elm)
						e.preventDefault()
						e.stopPropagation()
						// set selection to the last child of pv_elm
						let children = [...pv_elm.childNodes]
						let last_child = children[children.length-1]
						last_child = last_child.nodeName == '#text'||last_child.nodeName=='BR' ? last_child : last_child.childNodes[0] // TODO - handle br!
						console.log("LAST CHILD ... ", last_child, children)
						
						selection.setBaseAndExtent(last_child, last_child.textContent.length, last_child, last_child.textContent.length);
						return false
					}
				}
				
			}
			// down key
			if(e.keyCode == 40){ 
				// get index
				if (b_node == e_node){
					// move to the previous node
					console.log('B_INDEX ... ', b_index)
					console.log('E_INDEX ... ', e_index)
					console.log('ARR ELMS ... ', arr_elms)
					if(b_index == arr_elms.length-1 || (b_index == arr_elms.length-2 && arr_elms[arr_elms.length-1].tag == 'BR') || b_node == elm_node){
						let next_elm = elm_node.nextElementSibling
						console.log("FOCUS NEXT !!")
						if(next_elm && next_elm.isContentEditable){
							next_elm.focus()
							e.preventDefault()
							e.stopPropagation()
							return false
						}
					}
				}
			}
			// check ctrkey is pressed

			// check if key 8 is pressed and whether we need to merge the two divs! or delete the div!
			if(e.keyCode == 8){
				console.log('B_index --> ', b_index, b_node , ' Start __i ',start_i)
				if(start_i==0 && b_index==0){
					console.log("MERGE PREV")
					// remove div and merge with previous one if exist
					// maybe await settimeout!
					// get node from previous element
					let l_node_index 
					let l_node_end 
					let pv_elm = elm_node.previousElementSibling
					if(pv_elm && pv_elm.isContentEditable){
						if(!pv_elm.childNodes.length)
							pv_elm.focus()
						else{
							l_node_index = pv_elm.childNodes.length-1
							// console.log(pv_elm.childNodes.length)
							l_node_end = pv_elm.childNodes[pv_elm.childNodes.length-1].textContent.length
						} 
					}	
					
					dispatch('merge_prev', html)
					await (new Promise(r => setTimeout(r)))
					if(l_node_index){
						let l_node = pv_elm.childNodes[l_node_index]
						if(l_node.nodeName !== '#text' && l_node.nodeName !== 'BR'){
								l_node = l_node.childNodes[0]
						}
						selection.setBaseAndExtent(l_node, l_node_end, l_node, l_node_end);
					}
					e.preventDefault()
				}
			}
		}

		// TODO - split divs!
		if(e.keyCode == 13 && e.shiftKey == false){
			dispatch('enter')
			e.preventDefault()
			e.stopPropagation()
			return false
		}
		// shift key pressed add br to the table in the selected element!!
		// get index -- get text -- get parent span -- add br -- duplicate the span set cursor!
		if(e.keyCode == 13 && e.shiftKey == true){
			
			// e.preventDefault()
			// e.stopPropagation()
			
			let selection = window.getSelection()
			let b_node = selection.baseNode
					
			let b_index = getIndex(b_node)
			// let e_index = getIndex(e_node)
			let div_elm = b_node.nodeName != '#text' ? b_node.parentNode : b_node.parentNode.parentNode
			await (new Promise(r => setTimeout(r)))
			
			// not in rooot
			if(b_node.nodeName != "DIV" &&  ((b_node.parentNode && b_node.parentNode.tagName != 'DIV') || !['BR','#text'].includes(b_node.nodeName))){
				let parent = b_node.nodeName != '#text' ? b_node : b_node.parentNode
				//div_elm = parent.parentNode ? parent.parentNode : div_elm
				if(!parent.parentNode) {
					refresh()
					await (new Promise(r => setTimeout(r)))
					let p_elm = div_elm.childNodes[b_index]
					selection.setBaseAndExtent(p_elm, 0, p_elm, 0);
					return
				}
				// parent child text nodes
				let children = [...parent.childNodes]
				let elms = []
				
				for (let ch of children){
					if(ch && ch.textContent){
						elms.push({txt: ch.textContent, klass: arr_elms[b_index].klass, tag: parent.tagName})
					}else{
						elms.push({tag: "BR", txt: ""})
					}
				}
				
				// TODO - remove selected element!

				arr_elms.splice(b_index, 1,...elms)
				refresh()
				await (new Promise(r => setTimeout(r)))
				
				let p_elm = div_elm.childNodes[elms[0].tag == 'BR' ? b_index: b_index+2]
				console.log("P ELM --> ",b_index, div_elm.childNodes, p_elm)
				selection.setBaseAndExtent(p_elm, 0, p_elm, 0);
			}


		}

	}

	async function refresh(){

	 	let str = ''
	  arr_elms.forEach(elm => {
			if(elm.tag == 'BR'){
				str += '<br>'
			}else if(elm.tag == 'A'){
				str += `<a href=${elm.href} target='_blank' class="${elm.klass}">${elm.txt}</a>`
			}else if(elm.klass){
				str += `<span class="${elm.klass}">${elm.txt}</span>`
			}else{
				str += elm.txt
			}
		})
		
		html = str

	}

	let h_selection = null

	async function holdSelection(selection){
		if(h_selection) return
		//let selection = window.getSelection() 
		h_selection = {
			start_i: selection.baseOffset ,
			end_i: selection.extentOffset ,
			
			b_node: selection.baseNode,
			e_node: selection.extentNode
		}
		// console.log("H Selection ... ", h_selection)
		///await (new Promise(r => setTimeout(r)))
		
		///window.getSelection().removeAllRanges();
		//window.getSelection().setBaseAndExtent(h_selection.b_node, h_selection.start_i, h_selection.e_node, h_selection.end_i);

	}
	
	 
	async function setClass(class_name,link){
		
		// console.log("SETCLASS H Selection ... ", h_selection)

		arr_elms.forEach(e => delete e.selected)
		let selection = window.getSelection() 
		let selection_txt = selection.toString()

		let	start_i = h_selection ? h_selection.start_i : selection.baseOffset 
		let end_i = h_selection ? h_selection.end_i : selection.extentOffset 
		
		let b_node = h_selection ? h_selection.b_node : selection.baseNode
		let e_node = h_selection ? h_selection.e_node : selection.extentNode
		
		let b_index = getIndex(b_node)
		let e_index = getIndex(e_node)
		
		let same_node = b_node == e_node 
		 
		let reverse = b_index > e_index
		
		let sb_index = b_index
		let se_index = e_index
		if(reverse){
			sb_index = e_index
			se_index = b_index
			let x = start_i
			start_i = end_i
			end_i = x
		}
		
		let edit_node = b_node.parentNode
		if(b_node.parentNode.tagName !== 'DIV'){
			edit_node = edit_node.parentNode
		}
		
		let n_arr  = arr_elms.slice(sb_index,se_index+1)
			
		let arr1 = splitArr(n_arr,0,start_i, same_node && end_i)
		 
		let up_arr = arr1.length == 1 ? arr1 : arr1.length == 2&&(start_i==0||end_i==0) ? [arr1[0]] : [arr1[1]] 
		
		// toggleClass(up_arr1, class_name)
		n_arr.splice(0,1,...arr1)
		let arr2 = []
		if(!same_node){
			arr2 = splitArr(n_arr,n_arr.length-1, end_i, false) 
			n_arr.splice(n_arr.length-1,1,...arr2)
			up_arr = up_arr.concat(n_arr.slice(1,n_arr.length-(arr2.length == 1 ? 0:1)))  
		}
		// up_arr = up_arr.filter(Boolean)
		// up_arr = up_arr.map( a => (!a&&{txt: '',tag:'BR'})||a)
		toggleClass(up_arr, class_name,link)
		up_arr.forEach(e => e.selected= true)
		
		arr_elms.splice(sb_index,se_index-sb_index+1,...n_arr)

		let p_selector = {}
		mergeArr(p_selector)
		
		

		refresh()

		h_selection = null 

		await tick()

		let ch_nodes = [...edit_node.childNodes].filter(elm => elm.nodeName !== '#text' || (elm.nodeName == '#text' && elm.length>0))
		
		let start_node = ch_nodes[p_selector.a_start]
		let end_node = ch_nodes[p_selector.a_end]
		start_node = start_node.nodeName == '#text' ? start_node : start_node.firstChild
		end_node = end_node.nodeName == '#text' ? end_node : end_node.firstChild

		await (new Promise(r => setTimeout(r)))
		
		console.log("SELECTION .... ",start_node, p_selector.s_start, end_node, p_selector.s_end)
		window.getSelection().removeAllRanges();
		window.getSelection().setBaseAndExtent(start_node, p_selector.s_start, end_node, p_selector.s_end);
		
		await tick()
		holdSelection(window.getSelection())

		
	} 
	

	async function setGClass(klass){
		
		if(klass){
			if(reg_txt_size.test(klass)){
				
				gklass = gklass.replace(code_class,'').replace(quote_class,'').trim()
				replaceGClass(klass, reg_txt_size)
				// remove code and quote classes

			}else if( klass.startsWith('code')){
				// remove text & quote classes
				if(!gklass.includes(code_class)){
					gklass = gklass.replace(quote_class,'').replace(g_reg_txt_size,'').trim()
					gklass +=  ' '+code_class

				}
			
			}else if( klass.startsWith('quote')){
				// remove code & text classes!
				if(!gklass.includes(quote_class)){
					gklass = gklass.replace(code_class,'').replace(g_reg_txt_size,'').trim()
					gklass +=  ' '+quote_class
				}

			}else if (reg_position.test(klass)){
				replaceGClass(klass, reg_position)
			}else if (reg_leading.test(klass)){
				replaceGClass(klass, reg_leading)
			}else {
				toggleGClass(klass)
			} 
			 
		}else{
			elm.klass = klass
		}

	}
	
	function mergeArr(p_selector){
		// console.log("ARR _ELM ",arr_elms)
		let n_arr = [{...arr_elms[0]}]
		
		if(arr_elms[0].selected){
			// arr_elms[0].select_range = [0,arr_elms[0].txt.length-1]
			//arr_elms[0].select_start = 0
			p_selector.s_start = 0
			p_selector.a_start = 0
			p_selector.s_end = arr_elms[0].txt.length
			p_selector.a_end = 0
		}
		
		
		let f_arr_elms = arr_elms//.filter(e => !!e.txt)
		for(let i=1; i<f_arr_elms.length;i++){

				if(f_arr_elms[i].txt && f_arr_elms[i].tag == f_arr_elms[i-1].tag && f_arr_elms[i].klass == f_arr_elms[i-1].klass){
					
					if(f_arr_elms[i].selected && !f_arr_elms[i-1].selected){
						p_selector.s_start = n_arr[n_arr.length-1].txt.length
						p_selector.a_start = n_arr.length-1
						p_selector.s_end = n_arr[n_arr.length-1].txt.length+f_arr_elms[i].txt.length
						p_selector.a_end = n_arr.length-1
					}
				  
					n_arr[n_arr.length-1].txt += f_arr_elms[i].txt 
					

				}else{
					if(f_arr_elms[i].txt){
						n_arr.push({...f_arr_elms[i]})
						
						if(f_arr_elms[i].selected && !f_arr_elms[i-1].selected){
							p_selector.s_start = 0
							p_selector.a_start = n_arr.length-1
							p_selector.s_end = n_arr[n_arr.length-1].txt.length
							p_selector.a_end = n_arr.length-1
						}
						
					}else if(f_arr_elms[i].tag == 'BR'){
						n_arr.push({...f_arr_elms[i]})
					}
				}
				
				if(f_arr_elms[i].selected && f_arr_elms[i-1].selected){
					p_selector.s_end = n_arr[n_arr.length-1].txt.length
					p_selector.a_end = n_arr.length-1
				}
				// last_elm = f_arr_elms[i]
			
		}
		
	 // console.log("NARRRR ",n_arr)
		arr_elms = n_arr
	}

	function toggleColor(arr,klass){

		for(let elm of arr){
			if(elm.klass){
				let classes = elm.klass.split(' ')
				let s_color_index = classes.findIndex(c => klass.startsWith('text') ? reg_txt_color.test(c) : reg_bg_color.test(c))
				let selected_color_class = ~s_color_index ? classes[s_color_index] : ''
				if(selected_color_class){
					// remove old selected color
					elm.klass = elm.klass.replace(selected_color_class,'').trim()
				}
				elm.klass = elm.klass.split(' ').concat([klass]).join(' ') 
			}else{
				elm.klass = klass
			}
		}

	}


	let code_class = 'code text-sm font-mono px-8 py-6 bg-gray-200'
	let quote_class = 'quote text-xl border-l-4 border-gray-800 px-4 font-serif'



	function toggleGClass(klass){

		if(gklass.includes(klass)){
			gklass = gklass.replace(klass,'').trim()
		}else{
			gklass = gklass.split(' ').concat([klass]).join(' ')
		}

	}

	function replaceGClass(klass, reg){
		
		let classes = gklass.split(' ')
		let s_index = classes.findIndex(c => reg.test(c))
		let selected_class = ~s_index ? classes[s_index] : ''
		if(selected_class){
			gklass = gklass.replace(selected_class,'').trim()
		}
		gklass = gklass.split(' ').concat([klass]).join(' ')

	}

	
	let reg_txt_size = /^text\-(sm|base|xl|2xl|3xl)/
	let g_reg_txt_size = /text\-(sm|base|xl|2xl|3xl)/
	let reg_leading = /^leading\-(none|tight|snug|normal|relaxed|loose)/
	let reg_position = /^text\-(left|right|center)/
	let reg_txt_color = /^text\-(gray|red|orange|yellow|green|blue|indigo|purple|pink)/
	let reg_bg_color = /^bg\-(gray|red|orange|yellow|green|blue|indigo|purple|pink)/
	
	function toggleClass(arr, klass, link){

		if(reg_txt_color.test(klass) || reg_bg_color.test(klass)){
			toggleColor(arr,klass)
			return
		}

		if(arr.find(e => e.tag!='BR' && (!e.klass||!e.klass.includes(klass)))){
			// add class
			for(let elm of arr){
				if(elm.tag != 'BR' && link){
						elm.tag = 'A'
						elm.href = link
				}
				if(!elm.klass || !elm.klass.includes(klass)){
					elm.klass = elm.klass ? elm.klass.split(' ').concat([klass]).join(' ') : klass
				}
			}
		//else remove class	  
		}else{
			for(let elm of arr){
				if(elm.tag != "BR" && link){
					elm.tag = 'A'
					elm.href = link
				}
				if(!link && elm.klass && elm.klass.includes('link')){
					delete elm.href
					delete elm.tag
				}
				elm.klass = (elm.klass||'')&&elm.klass.replace(klass,'').trim()
				if(elm.klass == '')
					delete elm.klass
			}
		}

	}
	// a_i arr_index
	// s_i string_index
	function splitArr(arr, a_i, s_i, e_i){
		let start = s_i
		let end = e_i||arr[a_i].txt.length
		if(e_i && e_i<s_i){
			start = e_i 
			end = s_i 
		}  
		let s1 = arr[a_i].txt.slice(0,start)
		let s2 = arr[a_i].txt.slice(start,end)
		let arr1 = []
		let i = 0
		if(s1){
			arr1[i++] = {txt: s1, klass: arr[a_i].klass, tag: arr[a_i].tag, href: arr[a_i].href }
		}
		if(s2){
			arr1[i++] = {txt: s2, klass: arr[a_i].klass, tag: arr[a_i].tag, href: arr[a_i].href }
		}
		if(e_i && arr[a_i].txt.slice(end,arr[a_i].txt.length)){
			arr1[i++] = {txt: arr[a_i].txt.slice(end,arr[a_i].txt.length), klass: arr[a_i].klass, tag: arr[a_i].tag, href: arr[a_i].href}
		}
		 
		return arr1
	}
	
	function getIndex(node){
		let p_node = node
		if(node.nodeName == 'DIV') return 0
		if(['SPAN','BR','A'].includes(node.parentNode.tagName)){
			p_node = node.parentNode
		}
		return [...p_node.parentNode.childNodes].filter(n => n.nodeName!='#text' || n.length>0).indexOf(p_node)
		
	}
	
	// EVENT FN
	
	function extractClasses(b_index,e_index){
		if(b_index > e_index){
			let x = b_index
			b_index = e_index 
			e_index =  x 
		}
		let arr_slice = arr_elms.slice(b_index,e_index+1)
		if(!arr_slice[0].klass) 
			return ''

		let b_class = arr_slice[0].klass
		// find similar classes
		let arr_classes = b_class.split(' ')
		// if word not found remove it
		for(let i=1; i<arr_slice.length;i++){
			// get all the common classes!
			let elm = arr_slice[i]
			for(let w_class of arr_classes){
				if(elm.klass && !elm.klass.includes(w_class)){
					b_class = b_class.replace(w_class,'').trim()
				}
			}
		}
		return b_class
	}

	function extractLink(b_index,e_index){
		if(b_index > e_index){
			let x = b_index
			b_index = e_index 
			e_index =  x 
		}
		let arr_slice = arr_elms.slice(b_index,e_index+1)
		
		let href = ''

		for(let i=0; i<arr_slice.length;i++){
			// get all the common classes!
			let elm = arr_slice[i]
			if(elm.href){
				href = elm.href
				break
			}
		}
		
		return href
	}
	
	
	let l_selected = ''
	async function fireSelect(e){
		await (new Promise(r => setTimeout(r)))
		h_selection = null
		await (new Promise(r => setTimeout(r)))
		let selection = window.getSelection() 
		let selection_txt = selection.toString()
		if(selection_txt){				
			let b_node = selection.baseNode
			let e_node = selection.extentNode
			let b_index = getIndex(b_node)
			let e_index = getIndex(e_node)

			let base_node = b_index < e_index ? b_node : e_node
			holdSelection(selection)
			// extract classes to pass them to the toolbar!
			let classes = extractClasses(b_index,e_index)
			let href = extractLink(b_index,e_index)
			dispatch('select',{setClass, setGClass, base_node: b_node, classes, g_classes: gklass, href})
		}else{
			hideSelect()

		}
		l_selected = selection_txt

	}
	
	function hideSelect(){
		dispatch('hideselect')
	}

	
</script>

<div bind:innerHTML={html} placeholder='Type your text here' spellcheck="false" contenteditable="true" on:keydown={handleKeydown}  class="focus:outline-none {gklass}" on:mouseup={fireSelect} on:keyup={fireSelect} >
	
</div>
					  