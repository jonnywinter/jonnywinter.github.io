---
layout: post
title: "Meraki API & PowerShell - PUT & DELETE Requests"
author: "Jonny Winter"
categories: journal
tags: [API,Meraki,PowerShell]
image: Meraki-API-PowerShell-Code-2.png
comments: true
---

*"The difference between the POST and PUT APIs can be observed in request URIs. POST requests are made on resource collections, whereas PUT requests are made on a single resource." [...] "As the name applies, DELETE APIs are used to delete resources."* - [restfulapi.net](https://restfulapi.net/http-methods/#put)

## Summary

In my [last post]({{ site.github.url }}{% post_url 2021-01-11-meraki-api-powershell-post %}) we walked through the steps - and a few more - on how to perform a POST request to the Meraki Dashboard API to create a new network in PowerShell. In this post, we're going to perform both PUT & DELETE requests to update and then delete a resource - a network in our case.  

## Requirements

To keep it simple, we'll stick to the same requirements as the last post. For ease, here are those requirements again - 
<br>
*"To perform the majority of what we're going to do below there aren't many requirements other than having access to PowerShell, a [Meraki Dashboard Account](https://documentation.meraki.com/Getting_Started) and [generating your API key](https://documentation.meraki.com/General_Administration/Other_Topics/The_Cisco_Meraki_Dashboard_API). Simple stuff. Once you're at that point, read on. For reference, I'll be using [Windows PowerShell ISE](https://docs.microsoft.com/en-us/powershell/scripting/windows-powershell/ise/introducing-the-windows-powershell-ise?view=powershell-5.1) as it's built into Windows 10 and easy to use, but you could use another IDE - like [VScode](https://code.visualstudio.com/)."*

## My Environment

Coffee: [Decaf, Blend from Union Hand-Roasted Coffee ](https://unionroasted.com/collections/decaf/products/decaf-blend)
<br>
Music: [Clutch by Clutch](https://open.spotify.com/album/5snxt6YTFHrBD8ACd0hPJA?si=0p49Rd5kRjujYuMo-fpGhQ)
<br>
OS: Windows 10 Pro v20H2 x64.

## Tip o' the Hat

The Meraki [API docs](https://developer.cisco.com/meraki/api-v1/) on developer.cisco.com

## Let's begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**

Okie dokie. Let's get PowerShell ISE open and create the beginings of a script. In order to perform the PUT & DELETE requests, to modify and remove a resource on a server respectively, we are going to need two things - the ID of the organisation that the network belongs to, as well as the ID of that network. In the below text - which is largely formed of the script from the last post (and the one before that) - we have defined a few variables and using Invoke-RestMethod we can see a list of organisations and their IDs in a response, then taking an ID we can perform a second request to get a list of networks and their IDs replace {organizationId} with a valid organisation ID- 
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
###
#Back to the script
###
$networks = Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/{organizationId}/networks" -Headers $Headers
Out-Host -InputObject $networks
###
#Example output below (not script text)
###
id                   name          productTypes      
--                   ----          ------------      
L_1234567891         New Network 1 {switch, wireless}
N_1234567892         New Network 2 {switch}          
```
Select a network that you want to rename/delete, note down the coresponding ID and check out the instructions written by Meraki for [PUT](https://developer.cisco.com/meraki/api-v1/#!update-network) & [DELETE](https://developer.cisco.com/meraki/api-v1/#!delete-network). Lets tackle the PUT first, which is the request to update a resource. The documentation instructs us to send a PUT request to /networks/{networkId}. As with the POST request, we need to supply all data that we will be changing about the network in JSON format - a common theme with Meraki. There is a sample body on that link, but for ease I've included it below - 
```json
{
    "name": "Long Island Office",
    "timeZone": "America/Los_Angeles",
    "tags": [ "tag1", "tag2" ]
}
```
... but in this instance, we're simply going to rename the network - meaning our JSON body will only need one KVP (Key-value Pair); name. To do this, we will need to create the JSON body in PowerShell as the variable body then convert it into the JSON format using the ConvertTo-Json cmdlet, stored as another variable called jsonBody.
```powershell
$body = @{
    "name"              = "Renamed Network"
    }
$jsonBody = ConvertTo-Json -InputObject $body
```
For the request, we need to specify Put after -Method, supply the JSON formatted body using the -body switch followed by the variable with our JSON formatted body and replace {networkId} with your network ID from a few steps back and you're good to go. Like so - 
```powershell
Invoke-RestMethod -Method Put -Uri "https://api.meraki.com/api/v1/networks/{networkId}" -Headers $Headers -Body $jsonBody
```
Put together, with the response included, it'll look something like this - 
```powershell
$APIKey = "Enter your API key here"
$headers = @{
    "Content-Type" = "application/json"
    "X-Cisco-Meraki-API-Key" = $APIKey
}

$body = @{
    "name"              = "Renamed Network"
    }
$jsonBody = ConvertTo-Json -InputObject $body

Invoke-RestMethod -Method Put -Uri "https://api.meraki.com/api/v1/networks/{networkId}" -Headers $Headers -Body $jsonBody
Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/{organizationId}/networks" -Headers $Headers
###
#Example output below (not code text)
###
id               : L_Some-Network-ID
organizationId   : 1_Some-Organisation-ID
name             : Renamed Network
productTypes     : {switch, wireless}
timeZone         : America/Los_Angeles
tags             : {}
enrollmentString : 
url              : https://n1.meraki.com/Renamed-Network-swit/n/xxxxxxxx/manage/usage/list
notes            :
```
Mission accomplished. Very similar to the POST, just instead of sending the request to the organisation (collection) we send it directly to the network (resource) - as seen in the URI. In the same light, the DELETE request follows the exact same idea - send the request to the resource - although we don't have to supply a JSON formatted body. Simply change -Method Put to -Method Delete, delete the -body $jsonBody and you'll be good-to-go. Like so - 
```powershell
Invoke-RestMethod -Method Delete -Uri "https://api.meraki.com/api/v1/networks/{networkId}" -Headers $Headers
```
To make it a little more user friendly, having a person type the name of the network after typing the name of the organisation, you could end up with something like the below. The following example is, once saved, easy to run by right-clicking the file and selecting Run with PowerShell - 
```powershell
$APIKey = "Enter your API key here"
$headers = @{
    "Content-Type" = "application/json"
    "X-Cisco-Meraki-API-Key" = $APIKey
}
Write-Host "This script allows us to rename & delete networks within an organisation."
Write-Host ""

$organisations = Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations" -Headers $Headers
Out-Host -InputObject $organisations

$organisationName = read-host -Prompt "Please type an organisation name"
foreach ($organisation in $organisations){
    if ($organisation.name -like $organisationName.ToLower()) {
        $organisationId = $organisation.id
    }
}

$question1 = Read-Host -Prompt "Would you like to delete a network (y/n)"
if ($question1.ToLower() -eq "y"){
    $networks = Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/$($organisationId)/networks" -Headers $Headers
    $networks | Format-Table id,name,productTypes

    $networkName = read-host -Prompt "Please type a network name"
    foreach ($network in $networks){
        if ($network.name -like $networkName.ToLower()) {
            $networkId = $network.id
        }
    }
    Invoke-RestMethod -Method Delete -Uri "https://api.meraki.com/api/v1/networks/$($networkId)" -Headers $Headers
    $networks = Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/$($organisationId)/networks" -Headers $Headers
    $networks | Format-Table id,name,productTypes
}

pause
```
We are using Read-Host to get the user to type in text, | FormatTable to display certain data in a table, .ToLower() to convert a string to lower case, an if (a -eq b){do something} statement for logic and ConvertTo-Json to convert to JSON. Other than that, it's a very simple scipt - similar to the one in the last post. 

Using the PUT and DELETE methods explained above, you can start to build scripts to rename & delete things like SSIDs, VLANs, etc., etc. The destination you'll need to add to the base URI for each query are found on [https://developer.cisco.com/meraki/api-v1](https://developer.cisco.com/meraki/api-v1). 

Happy scripting!
