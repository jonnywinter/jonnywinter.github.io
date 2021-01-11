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

In my [last post]({{ site.github.url }}{% post_url 2021-01-10-meraki-api-powershell-get %}) we walked through the nesecary steps - and a few more - on how to perform a GET request to the Meraki Dashboard API and display data in a table in PowerShell. In this post, we're going to follow on from there and perform a POST request to create a resource - a network in our case. We'll then go a bit further and create a script that can be run, allowing you to create networks on demand across any organisation you have access to.

## Requirements

To keep it simple, we'll stick to the same requirements as the last post. For ease, here are those requirements again - 
<br>
*"To perform the majority of what we're going to do below there aren't many requirements other than having access to PowerShell, a [Meraki Dashboard Account](https://documentation.meraki.com/Getting_Started) and [generating your API key](https://documentation.meraki.com/General_Administration/Other_Topics/The_Cisco_Meraki_Dashboard_API). Simple stuff. Once you're at that point, read on. For reference, I'll be using [Windows PowerShell ISE](https://docs.microsoft.com/en-us/powershell/scripting/windows-powershell/ise/introducing-the-windows-powershell-ise?view=powershell-7.1) as it's built into Windows 10 and easy to use, but you could use another IDE - like [VScode](https://code.visualstudio.com/)."*

## My Environment

Coffee: [Décaféiné by Carte Noire](https://www.cartenoire.co.uk/en/shop/instant/decafeine-100g.html)
<br>
Music: [Everything Is A-OK by Violent Soho](https://open.spotify.com/album/4IayAjHP3LfFZZ79jetguT?si=U3pCXCMWRuqXOn3vComiJg)
<br>
OS: Windows 10 Pro v20H2 x64.

## Tip o' the Hat

The Meraki [API docs](https://developer.cisco.com/meraki/api-v1/) on developer.cisco.com
<br>
joshand's [response](https://community.meraki.com/t5/Developers-APIs/Powershell-POST-Script-Help/m-p/61542) on community.meraki.com

## Let's begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**

Right. Let's get PowerShell ISE open and create the beginings of a script. In the below text - which is largely using the script created in the last post - we have defined a few variables and using Invoke-RestMethod we can see the list of organisations and their IDs in a response 
```powershell
$APIKey = "Enter your API key here"
$headers = @{
    "Content-Type" = "application/json"
    "X-Cisco-Meraki-API-Key" = $APIKey
}
$organisations = Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations" -Headers $Headers
Out-Host -InputObject $organisations
###
#Example output below (not script text)
###
id                 name                   url                                                            
--                 ----                   ---                                                            
80009             Org 1              https://n3.meraki.com/o/4xxxxxxd/manage/organization/overview  
80019             Org 2              https://n1.meraki.com/o/Sxxxxxb/manage/organization/overview  
80005             Org 3              https://n1.meraki.com/o/Fxxxxxd/manage/organization/overview  
```


Happy scripting!
<a href="#"><img alt="Meraki API PowerShell Get Organisations" src="/assets/img/Meraki-API-PowerShell-Get-Organisations.png"/></a>
