---
layout: post
title: "Integrating the Umbrella Enforcement API with Google Assistant"
author: "Jonny Winter"
categories: journal
tags: [Python,Flask,Azure,Umbrella,API,IFTTT,Google]
image: Creating-VLANs-CSV-Meraki-API-Flask.png
comments: true
---

*"The Cisco Umbrella Enforcement API is designed to give technology partners the ability to send security events from their platform/service/appliance within a mutual customer’s environment to the Umbrella cloud for enforcement. You may also list the domains and delete individual domains from the list. All received events will be segmented by the mutual customer and used for future enforcement."* - [Enforcement API on docs.umbrella.com](https://docs.umbrella.com/enforcement-api/docs)

## Summary

After stumbling onto Salnikov Andrey's [article](https://medium.com/@salnikov.andrei/hey-google-enable-my-meraki-guest-wifi-cc49bc01bc73) on Medium, I got inspired to do something similar but with a twist. I wanted to use Google Assistant & APIs, but I wanted to use the [Cisco Umbrella](https://umbrella.cisco.com/) [Enforcement API](https://docs.umbrella.com/enforcement-api/docs) to block a site on command rather than Meraki, use [Azure App Services](https://azure.microsoft.com/en-gb/services/app-service/) instead of [PythonAnywhere](https://www.pythonanywhere.com/) and feed a variable between [IFTTT](https://ifttt.com/) & [Flask](https://flask.palletsprojects.com/) as opposed to a static webhook endpoint. In this post I'm going to detail the steps required to simulate the below video in your own environment whilst adding in contextual information where relevant -

<a href="https://youtu.be/Bx4H-a5RQd8"><img alt="Umbrella API & Google Assistant" src="https://youtu.be/Bx4H-a5RQd8"/></a>

## My Environment

Coffee: [Liberacion, Guatemala from Union Hand-Roasted Coffee](https://unionroasted.com/products/bobolink-brazil)
<br>
Music: [Blood Sugar Sex Magik by Red Hot Chili Peppers](https://open.spotify.com/album/30Perjew8HyGkdSmqguYyg?si=aDM7FNXZQaK1flIO74GiCw)
<br>
OS: Windows 10 Pro v20H2 x64. 
<br>
IDE: [Visual Studio Code v1.52.1](https://code.visualstudio.com/)
<br>
Browser: [Google Chrome v88](https://www.google.com/intl/en_uk/chrome/)
<br>
Speaker: [Sonos Beam](https://www.sonos.com/en-gb/shop/beam.html)

## Tip o' the Hat

The [webhook.site](https://webhook.site) website to create test webhook endpoints.
<br>
The [IFTTT](https://ifttt.com) service to create the custom Google Assistant integration.
<br>
The [Postman](https://www.postman.com/) desktop client to send test POST requests to Flask.
<br>
SearchingGood's [post](https://www.reddit.com/r/ifttt/comments/871r0i/example_of_a_working_json_body_in_webhooks_with/) on Reddit.
<br>
Chrivand's [Python file](https://github.com/chrivand/UmbrellaPythonSamples/blob/master/UmbrellaEnforcementPostRequest.py) on GitHub.
<br>
Cisco Umbrella's [reference guide](https://docs.umbrella.com/enforcement-api/reference) on the Enforcement API.
<br>
The Cisco DevNet [Umbrella](https://developer.cisco.com/umbrella/) page.
<br>
Azure-Samples [Python file](https://github.com/Azure-Samples/python-docs-hello-world/blob/master/app.py) on GitHub.
<br>
Microsoft's Python-Flask quickstart [guide](https://docs.microsoft.com/en-gb/azure/app-service/quickstart-python?tabs=bash&pivots=python-framework-flask) on microsoft.com.

## Let's Begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**

WORDS

Happy scripting!
