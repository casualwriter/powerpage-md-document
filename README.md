## Powerpage Document Framework

A simple document framework for using markdown as system documentation. 

Documentation is always a boring job for developer. When working on the document of ["Powerpage"](https://github.com/casualwriter/powerpage), 
there are some markdown files composed for github (e.g. README.md).
Thinking that it will be greated to make use of these markdown files as system documentation, 
so write this program for [Powerpage Documentation](https://pingshan-tech.com/powerpage/doc/)   

Actually, the program only has one file. i.e. [index.html](index.html), in **pure javascript without dependance**.

Just simply put the following file in a folder of web server, and everything is done.

1. index.html
2. README.md
3. interface.md
4. development.md
5. pp-document.md


### Screen preview

<img alt="screen preview" src="pp-md-document.gif" style="width:85%; padding:30px">

### User Guide

Below is the default setup for [Powerpage Documentaiton](https://pingshan-tech.com/powerpage/doc/).

```
<body onload="loadMdFile( location.href.split('?file=')[1]||'README.md', '<b>Contents</b>' )">
<div id=header>
  <span id=title>Powerpage <small>(documentation)</small></span>
  <span id=menu style="float:right; padding:12px">
    <button onclick="loadMdFile( 'README.md', this.innerText )">Home</button>
    <button onclick="loadMdFile( 'interface.md', this.innerText )">API</button>
    <button onclick="loadMdFile( 'development.md', this.innerText )">Development</button>
    <button onclick="loadMdFile( 'pp-document.md', this.innerText )">Document.md</button>
    <button onclick="loadMdFile( 'pp-md-editor', this.innerText )" disabled>Markdown-Editor</button>
    <button onclick="loadMdFile( 'pp-web-crawler.md', this.innerText )" disabled>Web-Crawler</button>
    <button onclick="loadMdFile( 'pp-db-report.md', this.innerText )" disabled>DB-Reports</button> 
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

* Start Flie  (i.e. <body onload="loadMdFile( location.href.split('?file=')[1]||`'README.md'`, '<b>Contents</b>' )"> )
* Page Title  (ie. `<span id=title>{page-title}</span>`)
* Menu Items  (i.e. `<button onclick="loadMdFile( '{markdown-file}', this.innerText )">{document-title}</button>)` )

then copy [index.html](index.html) with markdown documents to web server. that's ALL!

ps: a hidden function for developer. Press [Alt-S] will toggles page between normal and raw HTML.

 
### Simple Markdown Parser

The program include a simple markdown parse in pure javascaript. no dependance, reusable.

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

### Simple TOC with scrollspy

The program also has a function for simple TOC (table-of-content) with scrollspy feature in pure javascaript.

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

//=== scrollspy feature
document.getElementById('right-panel').onscroll = function () {
  var list = document.getElementById('left-panel').querySelectorAll('a')
  var divScroll = document.getElementById('right-panel').scrollTop - 10
  var divHeight = document.getElementById('right-panel').offsetHeight
  for (var i=0; i<list.length; i++) {
    var pos = document.getElementById(list[i].innerText).offsetTop - divScroll  
    list[i].style['font-weight'] = ( pos>0 && pos<divHeight ? 600 : 400 )
  }
}
```
 
### Modificaiton History
  
* 2021/10/05, v0.48, initial verison, minor revision from pp-document.html
* 2021/10/06, v0.50, cater url parameter, get filename from url.
* 2021/10/08, v0.60, add scrollspy feature.
* 2021/10/12, v0.64, refine of simple markdown parser 

