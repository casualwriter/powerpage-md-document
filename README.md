## Document by Markdown

A simple document framework for using markdown as system documentation. 

<img alt="screen preview" src="pp-md-document.gif" style="width:85%; padding:30px">


### Background

Documentation is always a boring job for developer. 
When working on the document of ["Powerpage"](https://github.com/casualwriter/powerpage), 
there are some markdown files composed for github (e.g. README.md).  
I would like to make use of these markdown files for the following purposes

1. Generage document from markdown for **web hosting**. e.g. github page: https://casualwriter.github.io/powerpage/   
2. **Directly run from github repository via CDN**.  e.g. rawgit.org: https://ghcdn.rawgit.org/casualwriter/powerpage-md-document/main/source/doc/index.html)   
3. run within [Powerpage](https://github.com/casualwriter/powerpage) with **Powerpage API enabled**.

### How it works

Actually, the program only has **one file** (i.e. [index.html](source/index.html)), in **pure javascript without any dependance**.

it will do the following

1. load markdown file by js function `loadMdFile(file,title)`
2. parse it into html by js function `simpleMarkdown(mdtext)`  
3. show in right-panel with TOC+scrollspy by js function `simpleTOC( srcDIV, toDiv, title )`


### Usage Guide

Please direct modify index.html to link up the top-menu to your markdown files
 
Below is the default setup for [Powerpage Documentation](https://casualwriter.github.io/powerpage/).

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

* Start Flie  (ie. ``<body onload="loadMdFile( location.href.split('?file=')[1]||'README.md', '<b>Contents</b>' )">`` )
* Page Title  (ie. ``<span id=title>{page-title}</span>``)
* Menu Items  (ie. ``<button onclick="loadMdFile( '{markdown-file}', this.innerText )">{document-title}</button>)``)

then copy [index.html](source/index.html) with markdown documents to web server. that's ALL!

ps: a hidden function for developer. Press [Alt-S] will toggles page between normal and raw HTML.

### Markdown Parser

Program includes a simple markdown parser in vanilla javascript. Source can be found at [index.html](source/index.html)

Document of **supported markdown syntax** can be found at https://casualwriter.github.io/powerpage/?file=pp-md-document.md#markdown-syntax


 
### Modificaiton History
  
* 2021/10/05, v0.48, initial verison, minor revision from pp-document.html
* 2021/10/06, v0.50, cater url parameter, get filename from url.
* 2021/10/08, v0.60, add scrollspy feature.
* 2021/10/12, v0.64, refine of simple markdown parser
* 2021/10/21, v0.67, minor fix on md parser and TOC 

