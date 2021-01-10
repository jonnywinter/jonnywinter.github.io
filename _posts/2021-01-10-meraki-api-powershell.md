---
layout: post
title: "Meraki API & PowerShell"
author: "Jonny Winter"
categories: journal
tags: [API,Meraki,PowerShell]
image: Meraki-API-PowerShell-Menu.png
---

First post! *punches air* Exciting. In this post I'm going to document the steps that you'll need to take to perform API calls to the Meraki Dashboard and format the JSON responses in PowerShell. To perform the majority of what we're going to do below there aren't many requirements other than having a [Meraki Dashboard Account](https://documentation.meraki.com/Getting_Started) and [generating your API key](https://documentation.meraki.com/General_Administration/Other_Topics/The_Cisco_Meraki_Dashboard_API). Simple stuff. Once you're at that point, read on - 

My Environment - 
<br>
~Coffee: [House Roast, Original Blend from Union Hand-Roasted Coffee](https://unionroasted.com/collections/our-favourite-coffees/products/house-roast-original-blend)
<br>
~Music: [To The 5 Boroughs by Beastie Boys](https://open.spotify.com/album/1yw6pIVYjbf9WoLiPkIPJv?si=hFBr1chPQ9GnD48pULQMaQ)
<br>
~OS: Windows 10 Pro v20H2 x64.


```powershell
Invoke-RestMethod -method Get -uri "http:"
```

```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```
