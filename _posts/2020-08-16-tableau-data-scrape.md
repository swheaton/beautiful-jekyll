---
layout: post
title: Scraping Data from Tableau
tags: [python, web scraping, scraping]
image: assets/img/tableau_scrape/tableau_logo.jpeg
---

## Scraping Tableau
[Web page scraping](https://en.wikipedia.org/wiki/Web_scraping) can be a great way to suck in data from multiple sites in an automated fashion, when no simple API portal or download mechanism exists. But of course in the pursuit of beautiful, modern web pages, there can be all different manners of presentation formats for data you want. One of the simplest to scrape is an [HTML table](https://www.w3schools.com/html/html_tables.asp) because you can utilize [pandas read_html()](https://pandas.pydata.org/pandas-docs/version/0.23.4/generated/pandas.read_html.html).

As things get fancier, though, scraping becomes much harder. Tools like [Tableau](https://www.tableau.com/) provide excellent, easy to use business intelligence and data visualization solutions for organizations ... but are a nightmare to the hopeful data scraper! Note that Tableau has a CSV download option but it can be disabled by the creator, and is also very difficult to use programmatically. I wanted to scrape data from some Tableau charts for a project and I couldn't figure it out at first - all questions at the time like [this Reddit post](https://www.reddit.com/r/tableau/comments/a5kxn8/scraping_tableau_data/) were basically unanswered. _After_ I figured it out for myself (go figure), a [Stack Overflow post](https://stackoverflow.com/questions/61962611/how-can-i-scrape-tooltips-value-from-a-tableau-graph-embedded-in-a-webpage/61976399#61976399) came out giving most of the steps. In this post, I'm going to expand on the answer to that SO question, which hopefully will provide more details for each step as well as discuss additional issues that may come up.

https://covid19.colorado.gov/data/case-data

https://stackoverflow.com/questions/62095206/how-to-scrape-a-public-tableau-dashboard
