---
layout: post
title: "Creating VLANs via CSV Using the Meraki API With Flask"
author: "Jonny Winter"
categories: journal
tags: [Python,Flask,AJAX,JavaScript,jQuery,Meraki,API]
image: Creating-VLANs-CSV-Meraki-API-Flask.png
comments: true
---

*"jQuery is a small JavaScript library commonly used to simplify working with the DOM and JavaScript in general. It is the perfect tool to make web applications more dynamic by exchanging JSON between server and client."* - [Armin Ronacher on The Pallets Projects](https://flask.palletsprojects.com/en/1.1.x/patterns/jquery/)

## Summary

Running [Flask](https://flask.palletsprojects.com/) for web apps is great - a few lines of code, that are ready to [copy & paste](https://palletsprojects.com/p/flask/) from the Pallets Project site and you're good to go. Add in a choice [bootstrap](https://getbootstrap.com/) CSS and within minutes the terible looking HTML web pages you made as a kid (let alone Myspace!) are a distant memory. However, more often than not you're going to need to work with data that is retrieved from that web page, often by way of a form or bulk upload - what's the best way to do it with Flask? For simple things we can use [query strings](https://en.wikipedia.org/wiki/Query_string) in the *?foo=bar* format, but for more intensive stuff we're going to want to pass [JSON](https://www.w3schools.com/whatis/whatis_json.asp) - think, a .CSV file uploaded to a web browser with a lot of data within could need to be converted to JSON to be worked with in Python. In this post I'm going to write a few methods of passing data and sending responses, like redirects, between Flask and the web browser. 

## My Environment

Coffee: [Bobolink, Brazil from Union Hand-Roasted Coffee](https://unionroasted.com/products/bobolink-brazil)
<br>
Music: [Tigercub](https://open.spotify.com/artist/6ekYAO2D1JkI58CF4uRRqw?si=JfKaaMIeR1CWf1JkXyrPrA)
<br>
OS: Windows 10 Pro v20H2 x64. 
<br>
IDE: [Visual Studio Code v1.52.1](https://code.visualstudio.com/)
<br>
Browser: [Google Chrome v88](https://www.google.com/intl/en_uk/chrome/)

## Tip o' the Hat

The [Flask documentation](https://flask.palletsprojects.com/en/1.1.x/) by Pallets
<br>
Madan Sapkota's [response](https://stackoverflow.com/questions/18118627/redirecting-after-ajax-post) on Stack Overflow
<br>
The AJAX with jQuery [documentation](https://flask.palletsprojects.com/en/1.1.x/patterns/jquery/) on The Pallets Projects.

## Let's Begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**


Happy scripting!
