## Powerpage Web Crawler

``Powerpage Web Crawler`` is a portable lightweight web crawler using [**Powerpage**](https://github.com/casualwriter/powerpage). 
 
It is a simple html/js application demonstrating developing application using [Powerpage](https://github.com/casualwriter/powerpage). 

`Powerpage Web Crawler` is powerful and easy-to-use web scrawler suitable for blog site crawling and offline-reading.

Just simply define below, for example

* `base-url` := `https://dev.to/casualwriter`  // the home page of favor blog site
* `index-pattern` := `none`  // RegExp of the url pattern of category page
* `page-pattern` := `/casualwriter/[a-z]` // RegExp of the url pattern of content page
* `content-css` := `#main-title h1, #article-body`  //css selector for blog content 

Program will
 
* crawl all category pages.
* find out all urls of content page. 
* crawl content for one page, or all pages. 
* save setting and links to database (support multiple sites)
* save content pages to local files.
* allow off-line reading from local files.


### Screen Preview
 
![](pp-web-crawler.gif)


### Installation & Run

* No installation is needed, Just download and run ``powerpage.exe``.
* The package is same as [Powerpage](https://github.com/casualwriter/powerpage), only ``powerpage.ini`` is revised.


### Source Code & files

It is single file js/html application ([pp-web-crawler.html](source/pp-web-crawler.html)) in about 350 lines.

* style sheet [pp-web-crawler.css](source/pp-web-crawler.css) is used to markup crawled content in your preference.
* ``pp-sample.mdb`` is MS Access DB to save site setting and links.

for the source code of **Powerpage**, please visit https://github.com/casualwriter/powerpage/tree/main/source/src
  
  
## Usage Guide


### Test Crawling

* Input the base url first.
* Click [Crawl Once] to crawl the base page 
* find out the pattent of category page (regexp)
* find out the pattent of content page (regexp)
* open Chrome and goto content page, find out the css selector for crawling content

### Start carwling

* Click [Crawl Once] to crawl base url once.
* Click [Crawl Max] to crawl category pages recursively. (depends on max-page setting).
* Click [Stop] to stop crawling process.
* Double-click on the list of category page, will crawl the page for links of content page.
* Double-click on the list of content page, will crawl content of this page, and show in right-panel.
* Single-Click on the list of content page, will show page from local file (if saved) 
* May continue finetune the "content" definition, then double-click to crawl the content page.
* In right-panel, click on [Save To File] will save crawl page to html file.

### Work with database

* Click [Load Sites] to show the list of sample sites.
* Click on a sample site to load the site setting only
* Doubleclick on sample site to crawled links from database.
* Click on [x] to delete site from database
* Click [Save Site] will save site setting with crawled links to database (sample.mdb)

If everything is tested fine, may click [Save All to File] to crawl all pages to html files.
  
  
## Modification History

* 2021/06/28, v0.30, first version.
* 2021/07/02, v0.35, add sample sites.
* 2021/07/09, v0.40, save links to db, and misc enhancement
* 2021/07/11, v0.50, work with database, work with local files, and misc enhancement
* 2021/07/15, v0.55, bug fixed, and misc enhancement.
* 2021/07/19, v0.56, minor fixed.
* 2021/11/02, v0.63, align to powerpage 0.63 

 