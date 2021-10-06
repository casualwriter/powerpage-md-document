## Powerpage Document Framework

A simple document framework for using markdown as system documentation. 

Compose document is always a boring job for developer. When working on the document of "Powerpage", there are some markdown files composed for github (e.g. README.md).
Thinking that it will be greated to make use of these markdown files to gernerate a web site for system documentation.

Actually, the program only has one file. i.e. [index.html](index.html), in **pure javascript without dependance**.

Just simply put the following file in a folder of web server, and everything is done.

1. index.html
2. README.md
3. interface.md
4. development.md
5. pp-document.md

**this is it!** [Powerpage Documentation](https://pingshan-tech.com/powerpage/doc/)

### Screen preview

<img alt="screen preview" src="pp-document.gif" style="width:85%; padding:30px">

### User Guide

Below is the default setup for [Powerpage Documentaiton](https://pingshan-tech.com/powerpage/doc/).

```
<body onload="loadMdFile('README.md', 'Overview' )">
<div id=header>
  <span id=title><mark>Powerpage <small>(documentation)</small></mark></span>
  <span id=menu style="float:right; padding:12px">
    <button onclick="loadMdFile('README.md',this.innerText)">Home</button>
    <button onclick="loadMdFile('interface.md',this.innerText)">API</button>
    <button onclick="loadMdFile('development.md',this.innerText)">Development</button>
    <button onclick="loadMdFile('pp-document.md')">Document.md</button>
    <button onclick="loadMdFile('pp-markdown.md')" disabled>Markdown-Editor</button>
    <button onclick="loadMdFile('pp-web-crawler.md')" disabled>Web-Crawler</button>
    <button onclick="loadMdFile('pp-db-report.md')" disabled>DB-Reports</button> 
    <button onclick="window.print()">Print</button> 
    <button style="display:none" onclick="toggleHTML()" accesskey=s>ShowHTML</button> 
  </span>
</div>
<div id=content>
  <div id="left-panel"></div>
  <div id="right-panel"></div>
</div>
</body>
```

Please setup the following items for your documentation site.

* Page Title  (ie. `<span id=title>{page-title}</span>`)
* Start page  (ie. `<body onload="loadMdFile( {start-markdownm-file}, {document-title} )">` )
* Top Menu Itme (i.e. `<button onclick="loadMdFile( {markdown-file}, this.innerText)">{document-title}</button>)` )

then copy the file with markdown documents to web server. that's ALL!


More, a hidden function for developer. Press [Alt-S] will toggles page between normal and raw HTML.

```
//=== toggle HTML in right-panel. (this is a hidden function for developer)
function toggleHTML() {
  var html = document.getElementById('right-panel').innerHTML
  if (html.substr(0,5)=='<xmp>') {
     document.getElementById('right-panel').innerHTML = html.substr(5, html.length-11)
  } else {
     document.getElementById('right-panel').innerHTML = '<xmp>' + html + '</xmp>' 
  }  
}
```

### Simple Markdown Parser

The program include a simple markdown parse in pure javascaript. no dependance, reuseable.

```
//=== simple markdown parser
function simpleMarkdown(mdText) {

  // first, handle syntax for code-block
  mdText = mdText.replace(/\r\n/g, '\n')
  mdText = mdText.replace(/\n~~~ *(.*?)\n([\s\S]*?)\n~~~/g, '<pre><code title="$1">$2</code></pre>' )
  mdText = mdText.replace(/\n``` *(.*?)\n([\s\S]*?)\n```/g, '<pre><code title="$1">$2</code></pre>' )
  
  // split by "pre>", skip for code-block and process normal text
  var mdHTML = ''
  var mdCode = mdText.split( 'pre>')

  for (var i=0; i<mdCode.length; i++) {
    if ( mdCode[i].substr(-2) == '</' ) {
      mdHTML += '<pre>' + mdCode[i] + 'pre>'
    } else {
      mdHTML += mdCode[i].replace(/(.*)<$/, '$1')
      	.replace(/^##### (.*?)\s*#*$/gm, '<h5>$1</h5>')
        .replace(/^#### (.*?)\s*#*$/gm, '<h4 id="$1">$1</h4>')
        .replace(/^### (.*?)\s*#*$/gm, '<h3 id="$1">$1</h3>')
        .replace(/^## (.*?)\s*#*$/gm, '<h2 id="$1">$1</h2>')
        .replace(/^# (.*?)\s*#*$/gm, '<h1 id="$1">$1</h1>')    
        .replace(/^-{3,}|^\_{3,}|^\*{3,}/gm, '<hr/>')    
        .replace(/``(.*?)``/gm, '<code>$1</code>' )
        .replace(/`(.*?)`/gm, '<code>$1</code>' )
      	.replace(/^\>> (.*$)/gm, '<blockquote><blockquote>$1</blockquote></blockquote>')
      	.replace(/^\> (.*$)/gm, '<blockquote>$1</blockquote>')
        .replace(/<\/blockquote\>\n<blockquote\>/g, '\n<br>' )
        .replace(/<\/blockquote\>\n<br\><blockquote\>/g, '\n<br>' )
      	.replace(/!\[(.*?)\]\((.*?) "(.*?)"\)/gm, '<img alt="$1" src="$2" $3 />')
      	.replace(/!\[(.*?)\]\((.*?)\)/gm, '<img alt="$1" src="$2" />')
      	.replace(/\[(.*?)\]\((.*?) "(.*?)"\)/gm, '<a href="$2" title="$3">$1</a>')
        .replace(/<http(.*?)\>/gm, '<a href="http$1">http$1</a>')
        .replace(/\[(.*?)\]\(\)/gm, '<a href="$1">$1</a>')
      	.replace(/\[(.*?)\]\((.*?)\)/gm, '<a href="$2">$1</a>')
        .replace(/^[\*|+|-][ |.](.*)/gm, '<ul><li>$1</li></ul>' ).replace(/<\/ul\>\n<ul\>/g, '\n' )
        .replace(/^\d[ |.](.*)/gm, '<ol><li>$1</li></ol>' ).replace(/<\/ol\>\n<ol\>/g, '\n' )
      	.replace(/\*\*\*(.*)\*\*\*/gm, '<b><em>$1</em></b>')
      	.replace(/\*\*(.*)\*\*/gm, '<b>$1</b>')
      	.replace(/\*([\w \d]*)\*/gm, '<em>$1</em>')
        .replace(/___(.*)___/gm, '<b><em>$1</em></b>')
        .replace(/__(.*)__/gm, '<u>$1</u>')
        .replace(/_([\w \d]*)_/gm, '<em>$1</em>')
      	.replace(/~~(.*)~~/gm, '<del>$1</del>')
        .replace(/\^\^(.*)\^\^/gm, '<ins>$1</ins>')
        .replace(/ +\n/g, '\n<br/>')
        .replace(/\n\s*\n/g, '\n<p>\n')
      	.replace(/^ {4,10}(.*)/gm, '<pre><code>$1</code></pre>' )
      	.replace(/^\t(.*)/gm, '<pre><code>$1</code></pre>' )
      	.replace(/<\/code\><\/pre\>\n<pre\><code\>/g, '\n' )
      	.replace(/\\([`_\\\*\+\-\.\(\)\[\]\{\}])/gm, '$1' )
  	}  
  }
  
  return mdHTML.trim()
}
```

### Simple TOC (table of content)

The program also has a function for simple TOC (table-of-content) in pure javascaript. no dependance, reuseable.

```
//=== simpleTOC: show Table of Content
function simpleTOC( title, srcDiv, toDiv ) {

  var toc = document.getElementById(srcDiv||'right-panel').querySelectorAll('h2,h3')
  var html = '<h4> ' + (title||'Content') + '</h4><ul id="toc">';

  for (var i=0; i<toc.length; i++ ) {
  	if (!toc[i].id) toc[i].id = "toc-item-" + i;
    
  	if (toc[i].nodeName === "H2") {
  		html += '<li style="background:#f6f6f6"><a href="#' + toc[i].id + '">' + toc[i].textContent + '</a></li>';
  	} else if (toc[i].nodeName === "H3") {
  		html += '<li style="margin-left:12px"><a href="#' + toc[i].id + '">' + toc[i].textContent + '</a></li>';
  	} else if (toc[i].nodeName === "H4") {
  		html += '<li style="margin-left:24px"><a href="#' + toc[i].id + '">' + toc[i].textContent + '</a></li>';
  	}
  }

  document.getElementById(toDiv||'left-panel').innerHTML = html   
}
```
 
### Modificaiton History
  
* 2021/10/06, v0.50, initial verison, minor revision from pp-document.html

