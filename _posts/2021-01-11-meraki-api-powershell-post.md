---
layout: post
title: "Meraki API & PowerShell - POST Requests"
author: "Jonny Winter"
categories: journal
tags: [API,Meraki,PowerShell]
image: Meraki-API-PowerShell-Code.png
---

*"Verbs in the API follow the usual REST conventions: GET returns the value of a resource or a list of resources, depending on whether an identifier is specified. POST adds a new resource. PUT updates a resource. DELETE removes a resource."* - [Cisco Meraki](https://documentation.meraki.com/General_Administration/Other_Topics/The_Cisco_Meraki_Dashboard_API#API_Requests)

## Summary

Using the Meraki Dashboard REST API we can use CRUD (**C**reate, **R**ead, **U**pdate & **D**elete) to interact with the server in a programatic way, as opposed to using the web UI. In this post I'm going to show the steps that are needed in order to perform API calls to the Meraki dashboard and display the JSON responses in PowerShell. We'll then go a bit further and create a script that can be double clicked to present you the list of networks against an organisation, using a user input. 

## Requirements

To perform the majority of what we're going to do below there aren't many requirements other than having access to PowerShell, a [Meraki Dashboard Account](https://documentation.meraki.com/Getting_Started) and [generating your API key](https://documentation.meraki.com/General_Administration/Other_Topics/The_Cisco_Meraki_Dashboard_API). Simple stuff. Once you're at that point, read on. For reference, I'll be using [Windows PowerShell ISE](https://docs.microsoft.com/en-us/powershell/scripting/windows-powershell/ise/introducing-the-windows-powershell-ise?view=powershell-7.1) as it's built into Windows 10 and easy to use, but you could use another IDE - like [VScode](https://code.visualstudio.com/). 

## My Environment

Coffee: [Décaféiné by Carte Noire](https://www.cartenoire.co.uk/en/shop/instant/decafeine-100g.html)
<br>
Music: [Everything Is A-OK by Violent Soho](https://open.spotify.com/album/4IayAjHP3LfFZZ79jetguT?si=U3pCXCMWRuqXOn3vComiJg)
<br>
<iframe src="https://open.spotify.com/embed/album/4IayAjHP3LfFZZ79jetguT" width="300" height="380" frameborder="0" allowtransparency="true" allow="encrypted-media"></iframe>
OS: Windows 10 Pro v20H2 x64.

## Tip o' the Hat

The Meraki [API docs](https://developer.cisco.com/meraki/api-v1/) on developer.cisco.com
<br>
NetworkEngineer7's [video](https://www.youtube.com/watch?v=MTOyge6ZZmg&ab_channel=NetworkEngineer7) on YouTube
<br>
relxteb's Meraki [GitHub repository](https://github.com/relaxteb/Meraki)

## Let's begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**

Now you've generated your API key (and created a dashboard account if you do not already have one to use), get PowerShell ISE open and we can go about calling the Dashboard [REST API](https://www.youtube.com/watch?v=7YcW25PHnAA&t=1s&ab_channel=WebConcepts). To get started, we are going to perform a [GET request](https://www.youtube.com/watch?v=guYMSP7JVTA&ab_channel=Telusko) to fetch all of the organisations in which your dashboard administrator account has access to - if you have access to the [MSP Portal](https://documentation.meraki.com/General_Administration/Organizations_and_Networks/Using_the_MSP_Portal_to_Manage_Multiple_Organizations) you will return multiple organisations, if not then you will only return one.

In PowerShell, we're going to be using the [Invoke-RestMethod](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-restmethod?view=powershell-7.1) cmdlet to interact with the Meraki Dashboard and storing the returned values in a [variable](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_variables?view=powershell-7.1). Once these variables have a value we will then display the values in a nicely formatted table.



Happy scripting!
<a href="#"><img alt="Meraki API PowerShell Get Organisations" src="/assets/img/Meraki-API-PowerShell-Get-Organisations.png"/></a>
