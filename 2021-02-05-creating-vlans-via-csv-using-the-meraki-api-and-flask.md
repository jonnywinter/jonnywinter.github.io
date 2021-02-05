---
layout: post
title: "Creating VLANs via CSV Using the Meraki API and Flask"
author: "Jonny Winter"
categories: journal
tags: [Python,Flask,AJAX,JavaScript,jQuery,Meraki,API]
image: Creating-VLANs-CSV-Meraki-API-Flask.png
comments: true
---

*"Meraki Dashboard API - A RESTful API to programmatically manage and monitor Meraki networks at scale. What can you do with it? Add new organizations, admins, networks, devices, VLANs, and more."* - [Meraki API Docs on developer.cisco.com](https://developer.cisco.com/meraki/api-v1/)

## Summary

Over the last [two](https://jonathan-winter.co.uk/journal/python-flask-iis.html) [posts](https://jonathan-winter.co.uk/journal/passing-data-between-html-flask.html), I've documented some how-to's using the almighty [Flask](https://flask.palletsprojects.com/en/1.1.x/), the Python package. However, in this post I'm going to document a project - this project is a small Flask web app that enables you to bulk create MX VLANs using the Meraki API. With minor alteration this could be used to iterate over hundreds of networks' worth of MXs, or be ammended to create firewall rules, etc., etc. To show the tool in action, here's a GIF of it working on my PC - 

<a href="#"><img alt="Adding a boilerplate to HTML" src="/assets/img/Meraki-VLAN-CSV-Tool.gif"/></a>

## My Environment

Coffee: [Bobolink, Brazil from Union Hand-Roasted Coffee](https://unionroasted.com/products/bobolink-brazil)
<br>
Music: [Paul's Boutique by Beastie Boys](https://open.spotify.com/album/1kmyirVya5fRxdjsPFDM05?si=zLgvHUIpReuGl322j89bWQ)
<br>
OS: Windows 10 Pro v20H2 x64. 
<br>
IDE: [Visual Studio Code v1.52.1](https://code.visualstudio.com/)
<br>
Browser: [Google Chrome v88](https://www.google.com/intl/en_uk/chrome/)

## Tip o' the Hat

Manav Kothari's [response](https://stackoverflow.com/questions/38636218/how-can-i-convert-csv-to-json-file-in-that-form/38636684#38636684) on Stack Overflow
<br>
Mudassar Ahmed Khan's [post](https://www.aspsnippets.com/Articles/Read-Convert-CSV-File-to-JSON-Array-in-jQuery-using-HTML5-File-API.aspx) on ASP Snippets
<br>
This TechSlides [article](http://techslides.com/convert-csv-to-json-in-javascript)
<br>
Espoir Murhabazi's [response](https://stackoverflow.com/questions/47627035/how-to-get-ajax-posted-json-in-flask) on Stack Overflow
<br>
The AJAX with jQuery [documentation](https://flask.palletsprojects.com/en/1.1.x/patterns/jquery/) on The Pallets Projects.

## Let's Begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**


Happy scripting!
