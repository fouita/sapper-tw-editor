<script>
    import hljs from 'highlight.js/lib/core';
    import {onMount} from 'svelte'
    import xml from 'highlight.js/lib/languages/xml'
    import 'highlight.js/styles/atom-one-dark-reasonable.css';
    hljs.registerLanguage('xml', xml);
    let b_str=""
    //let str="<div>Hello!</div><div class=''>sdklfjh skdjfh sdfs<br>sdfls kdfjsldk jsldf jlskdjsldkfl skdjfl ksdflksjdf lskdjfl skdfj slkjzelrkzerlzkjerlzkerlzkej rlzk jlze rlzkejr lkzelrkj<br>sdfksjdfh sdf<br>sdfkjsd<span class=\"font-mono px-2 text-gray-700 border border-gray-200 bg-gray-200\">fhsdf</span><br><span class=\"font-mono px-2 text-gray-700 border border-gray-200 bg-gray-200\">dskfjshdfkjsdf</span><br>zerlzkerjzelr</div><div class=''>sdfsdfs&nbsp;</div><div>Hello!</div><div class=''>sdklfjh skdjfh sdfs<br>sdfls kdfjsldk jsldf jlskdjsldkfl skdjfl ksdflksjdf lskdjfl skdfj slkjzelrkzerlzkjerlzkerlzkej rlzk jlze rlzkejr lkzelrkj<br>sdfksjdfh sdf<br>sdfkjsd<span class=\"font-mono px-2 text-gray-700 border border-gray-200 bg-gray-200\">fhsdf</span><br><span class=\"font-mono px-2 text-gray-700 border border-gray-200 bg-gray-200\">dskfjshdfkjsdf</span><br>zerlzkerjzelr</div><div class=''>sdfsdfs&nbsp;</div>"	
    
    export let str = ""
    let empty = true
    
    $: if (str!=""){empty=false}else{empty=true}
    
    $: if(mounted) {
        console.log("STR ",  str)
        b_str = Beautify(str)
    }
    
    let mounted = false
    
    onMount(()=>{
        b_str = Beautify(str)
        mounted = true
    })

    function Beautify(str) {
    var div = document.createElement('div');
    div.innerHTML = str.trim();

    return format(div, 0).innerHTML
    }

    function format(node, level) {
    var indentBefore = new Array(level++ + 1).join('  '),
        indentAfter = new Array(level - 1).join('  '),
        textNode;

    for (var i = 0; i < node.children.length; i++) {
        textNode = document.createTextNode('\n' + indentBefore);
        node.insertBefore(textNode, node.children[i]);
        
        
        
        format(node.children[i], level);

        if (node.lastElementChild == node.children[i]) {
        textNode = document.createTextNode('\n' + indentAfter);
        node.appendChild(textNode);
        
        }
    }

  return node;
}
let pre = null
let but = null
let copied = false

function SetPre(n){
    pre = n
}
function SetBut(n){
    but = n
    but.innerHTML="Copy"
}

function copyFunction() {
  const copyText = pre.textContent;
  const textArea = document.createElement('textarea');
  textArea.textContent = copyText;
  document.body.append(textArea);
  textArea.select();
  document.execCommand("copy");
  textArea.remove()
  but.innerHTML="Copied.."
  setTimeout(()=>but.innerHTML="Copy",5000)
  copied =true
  setTimeout(()=>copied=false,5000)
}

</script>


<div class="bg-gray-800 p-6 relative rounded overflow-auto">
    <div class="{empty? 'hidden':''} {copied? 'text-gray-500':'text-blue-300'} {copied? '':'cursor-pointer'} absolute sticky top-0 right-0 hover:{copied? '':'underline'} hover:{copied? '':'text-blue-200'} mb-4" on:click="{()=>copyFunction()}" use:SetBut>Copy</div>
    <pre class="max-h-screen">
        <code class="bg-gray-800 text-white " use:SetPre>{@html hljs.highlight('xml', b_str).value}</code>  
    </pre>
</div>
