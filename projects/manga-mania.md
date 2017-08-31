---
layout: project
title: Manga Mania
permalink: manga-mania/
---

### What is Manga Mania?

Manga-Mania is a simple desktop application for reading manga scraped from [MangaFox](http://mangafox.me).
It is written in Python3, with the generous aid of [PyQt5](https://sourceforge.net/projects/pyqt/),
[requests](http://docs.python-requests.org/en/master/) and [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/).

#### Features

* Advanced search functionality of MangaFox.
* Read chapters sequentially/jump to particular pages.
<br>

### Screenshots
[![Manga-Mania-1]({{site.url}}/resources/images/manga-mania-1-thumb.jpg "Search screen demo")]({{site.url}}/resources/images/manga-mania-1.png)
[![Manga-Mania-2]({{site.url}}/resources/images/manga-mania-2-thumb.jpg "Reading a manga")]({{site.url}}/resources/images/manga-mania-2.jpg)
<br>

### Source & Running Manga Mania

Manga Mania has a few dependencies:

* PyQt5
* requests
* BeautifulSoup4

You can easily install them using pip like so:

`$ pip install pyqt5 requests beautifulsoup4`

To run the Manga Mania, clone the repo and switch to the `manga-mania` directory and run `manga-mania.py`

```
$ git clone https://github.com/TheIllusionistMirage/Manga-Mania
$ cd Manga-Mania/manga-mania
$ python3 manga-mania.py
```
<br>

### Legal

I am neither affiliated to the manga sites mentioned in this application,
nor do I own any kind of rights to the manga these sites contain. I merely
provide a medium to access from your desktop what is already reachable using
a web browser. In no event shall I be held responsible for any kinds of damages,
physical or intellectual, arising from the use of Manga Mania. See
[LICENSE](https://github.com/TheIllusionistMirage/Manga-Mania/blob/master/LICENSE)
for detailed licensing info.
