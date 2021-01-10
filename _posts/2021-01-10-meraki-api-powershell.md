---
layout: post
title: "Meraki API & PowerShell"
author: "Jonny Winter"
categories: journal
tags: [API,Meraki,PowerShell]
image: Meraki-API-PowerShell-Menu.png
---

## Summary

To quote [Meraki](https://documentation.meraki.com/Getting_Started#:~:text=The%20Meraki%20dashboard%20itself%20is,organizations%2C%20networks%2C%20and%20devices), *"The Meraki dashboard itself is a centralized, web browser-based tool used to monitor and configure Meraki devices and services. A dashboard account is what you use to log in to the dashboard in order to manage and configure your organizations, networks, and devices."* Using the Meraki Dashboard REST API we can use CRUD (Create, Read, Update & Delete) to interact with the server in a programatic way, as opposed to using the web UI. In this post I'm going to show the steps that are needed in order to perform API calls to the Meraki dashboard and display the JSON responses in PowerShell. 

## Requirements

To perform the majority of what we're going to do below there aren't many requirements other than having access to PowerShell, a [Meraki Dashboard Account](https://documentation.meraki.com/Getting_Started) and [generating your API key](https://documentation.meraki.com/General_Administration/Other_Topics/The_Cisco_Meraki_Dashboard_API). Simple stuff. Once you're at that point, read on. For reference, I'll be using [Windows PowerShell ISE](https://docs.microsoft.com/en-us/powershell/scripting/windows-powershell/ise/introducing-the-windows-powershell-ise?view=powershell-7.1) as it's built into Windows 10 and easy to use. 

## My Environment

Coffee: [House Roast, Original Blend from Union Hand-Roasted Coffee](https://unionroasted.com/collections/our-favourite-coffees/products/house-roast-original-blend)
<br>
Music: [To The 5 Boroughs by Beastie Boys](https://open.spotify.com/album/1yw6pIVYjbf9WoLiPkIPJv?si=hFBr1chPQ9GnD48pULQMaQ)
<br>
OS: Windows 10 Pro v20H2 x64.

## Hat Tips

The Meraki [API docs](https://developer.cisco.com/meraki/api-v1/) on developer.cisco.com
<br>
NetworkEngineer7's [video](https://www.youtube.com/watch?v=MTOyge6ZZmg&ab_channel=NetworkEngineer7) on YouTube
<br>
relxteb's Meraki [GitHub repository](https://github.com/relaxteb/Meraki)

## TL;DR;

To understand what is happening here please fully read the article, however this is the first bit of code we're going to write so we can programatically get a list of all organisations we have access to.
<br>
```powershell
$APIKey = 'Enter your API key here'
$headers = @{
    'Content-Type' = 'application/json'
    'X-Cisco-Meraki-API-Key' = $APIKey
}
Invoke-RestMethod -Method Get -Uri 'https://api.meraki.com/api/v1/organizations' -Headers $headers
```
... the above output will be displayed in a PowerShell table. For those wanting a JSON output we can use the following text instead (with the $APIKey & $headers variables from the code snippet).
```powershell
$var = Invoke-RestMethod -Method Get -Uri 'https://api.meraki.com/api/v1/organizations' -Headers $headers
ConvertTo-Json -InputObject $var
```
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```
