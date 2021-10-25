# Development Guide

Powerpage is a ready-make Electron-like solution. No install, no compile, no packing. Just open editor to start coding.

Powerpage is very simple. It is a window with MS webbrowser control (equivalent to IE11). 
HTML Page talk to Powerpage when there is any additional requirement which web-browser cannot handle.

For example, 

* read a file, call ``javascript:pb.file.read()``
* run a program, call ``javascript:pb.run()``
* run sql to get data. call ``javascript:pb.db.query(sql,callback)``
* execute sql to update db,call ``javascript:pb.db.execute(sql,callback)``

-------------------------------------------------------------------
## Intreface / API

Powerpage provides "pb protocol command" to talk to html pages. 
When html page loaded, javascript object ``pb`` is provided as interface service provider.

for details, please refer to [interface  guide](interface.md).


-------------------------------------------------------------------
## Start page

Start page can be defined by commandline, or ini. Powerpage load the start html by the following sequence

1. from commandline. ``powerpage.exe your-start-page.html``

2. from powerpage.ini if no commandline option is found
```
[system]
start=your-start-page.html
```

3. if not defined in ini file, by default, powerpage load index.html if found or load powerpage.html if found. 

-------------------------------------------------------------------
## Command Line

Beside running javascript applications, Powerpage has wide usage by using commandline parameters.

~~~
powerpage.exe /ini={ini-file} /url={start-url}  /script={script-file} /fullscreen /print /silent
              /save={save-html} /pdf={output-pdf-file} /select={selector} /delay={1000}
~~~

* `/ini={ini-file}` specifies ini setting file. Aplication could be changed by change the ini file.
* `/url={start-url}` is used to specify startup link. Aplication could be changed by change startup link.
* `/script={script-file}` will specify user-defined javascript instead of `powerpage.js`. useful for js injection. 
* `/fullscreen` or `/kiosk` will run in fullscreen mode, useful for kiosk, or display board.
* `/silent` will run in silent mode (i.e. suppress js error message)
* `/print`` will load startup url, print and close program.
* `/save={save-html}` will load startup url, save to html file, and close program.  Useful for web-crawler
* `/pdf={output-pdf-file}` will load startup url, generate PDF file, and close program. useful for PDF generation.
* `/delay={1000}` specifies delay time (by milliseconds) for print/save/pdf options
* `/select={css-selector}` is applied for **print/save** to select part of html elements.  Useful for web-crawler

### Samples of using command-line

**General Usage**
* ``powerpage.exe /ini=pp-md-editor.ini`` run "Powerpage Markdown Editor" with its config ini
* ``powerpage.exe /url=pp-md-editor.html`` run "Powerpage Markdown Editor"
* ``powerpage.exe /url=pp-web-crawler.html`` run "Powerpage Web Crawler"
* ``powerpage.exe /url=pp-kanban.html /fullscreen`` run Kanban display board in fullscreen mode
* ``powerpage.exe /url=pp-md-document.html`` open "Powerpage Documents"
* ``powerpage.exe /url=facebook.com`` /script=myfacebook.js`` inject js script for facebook.com

**print page or save to html/pdf**
* ``powerpage.exe /url=http://haodoo.net/ /print`` print page of haodoo.net
* ``powerpage.exe /url=http://haodoo.net/ /pdf=haodoo.pdf`` save the page of haodoo.net to PDF file
* ``powerpage.exe /url=http://haodoo.net/ /save=haodoo.html`` save page "haodoo.net" to haodoo.html

**save web content to file (whole page or select by css-selecotr)**
* ``powerpage.exe /url=https://pingshan-tech.com/powerpage/doc /save=README.html`` save powerpage README (whole page)
* ``powerpage.exe /url=https://pingshan-tech.com/powerpage/doc /save=README.html /select=#content`` save powerpage README (#content:outerHTML)
* ``powerpage.exe /url=https://pingshan-tech.com/powerpage/doc /save=README.html /select=@#content`` save powerpage README (#content:innerText)
* ``powerpage.exe /url=https://pingshan-tech.com/powerpage/doc /save=README.html /select=#right-panel`` save powerpage README (#right-panel)

**save github content to html/pdf (select==.markdown-body)**
* ``powerpage /url=https://github.com/casualwriter/powerpage /save=README.html /select=.markdown-body`` to save README from github (has error msg)
* ``powerpage /url=https://github.com/casualwriter/powerpage /save=README.html /select=.markdown-body /silent`` to save README from github (silent mode)
* ``powerpage /url=https://github.com/casualwriter/powerpage /pdf=README.pdf /select=.markdown-body /silent`` to save in PDF format

  
-------------------------------------------------------------------
## INI Setting

The following setting can be customized for your application.

~~~
[system]
start   = app-start-page.html
script  = powerpage.js
version = version-info-of-about-dialog
credit  = copyright-info-of-about-dialog
about   = brief-description-of-applicaiton
github  = url-of-github
home    = url-of-app-home
title   = fix-title ( or [html] for show html title, [file]to show url)
extLibrary=Powerbuilder Extend Library (e.g. powerExt1.pbl,powerExt2.pbl)

[database]
DBMS      = ODBC | O90 | etc..
DbParm    = db-connection-parameter
SeverName = db-server-name (or @encrypted-string)
LogId     = db-login-id (or @encrypted-string)
LogPass   = db-login-passowrd (or @encrypted-string)

[browser]
title  = [html] | file | {fix-title} 
button = ABC  (a-about, b-goback, c-console)
status = [yes] | pure | noweb | hide | no
status.backcolor=status-background-color (default=67108864)
status.textcolor=status-text-color (default=0)
left  = left-position
right = right-position
width = window-width
height= window-height
~~~

-------------------------------------------------------------------
## Mini Buttons

* [A] - About dialog
* [B] - goback()
* [C] - Console
* [E] - Edit
* [R] - Reload page
* [W] - Openin web browser


-------------------------------------------------------------------
## Samples

Several sample applications are provided to demonstrated the powerpage application:

* [Markdown editor](https://github.com/casualwriter/powerpage-md-editor)
* [Web Crawler](https://github.com/casualwriter/powerpage-web-crawler)
* Database Browser
* kanBan
* Display Board
* PowerPage-Monkey 
* Application Framework

## Senario

to-be-continue...


