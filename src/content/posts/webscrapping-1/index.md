---
title: 'Getting Started with Web Scraping: Scraping Data from Static Sites'
published: 2024-10-20
description: ''
image: './Pasted image 20241020164045.png'
tags: ['webscraping']
category: 'DEV'
draft: false 
---

In this blog, we'll explore how to scrape data from static sites. But first, what are static sites? Static sites are generally non-interactive and return simple HTML rather than making various API calls. Most static sites, such as directories, contain data that stays unchanged over time. Wikipedia and Hacker News are great examples of static sites, and many WordPress sites also fall into this category. The key feature is that the data doesn't change dynamically—it remains the same as it was when initially loaded.

![alt text](<Pasted image 20241020164045.png>)

When you inspect the network calls for a static site, you'll notice that the entire data set is sent to the UI in a single call as an HTML file. Scraping a static site is relatively easy; the tricky part is filtering out the specific data you need.

All you have to do is identify the endpoint from which the data is fetched. Using your system's terminal, you can call this endpoint using the curl command to get the raw HTML data. We'll also explore how to achieve this using Python.

![alt text](<Pasted image 20241020164737.png>)

In the image above, you can see how the HTML file is returned. The next step is to filter the data you want. You can use regular expressions or other methods to extract only the relevant information. For instance, you can even ask ChatGPT to generate a regular expression that extracts only the URLs from a website.

Here's an example using curl:

```bash
curl -s https://en.wikipedia.org/wiki/Nissan_Skyline_GT-R | grep -oP '(?<=href=")[^"]*' | grep -E "^http"
```

![alt text](<Pasted image 20241020165205.png>)

This is just one way to filter the data, and there are many other techniques you can use to focus only on the information you need.