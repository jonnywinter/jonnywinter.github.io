---
layout: post
title: "Meraki API & PowerShell - PUT & DELETE Requests"
author: "Jonny Winter"
categories: journal
tags: [API,Meraki,PowerShell]
image: Meraki-API-PowerShell-Code.png
---

*"The difference between the POST and PUT APIs can be observed in request URIs. POST requests are made on resource collections, whereas PUT requests are made on a single resource." [...] "As the name applies, DELETE APIs are used to delete resources."* - [restfulapi.net](https://restfulapi.net/http-methods/#put)

## Summary

In my [last post]({{ site.github.url }}{% post_url 2021-01-10-meraki-api-powershell-post %}) we walked through the steps - and a few more - on how to perform a POST request to the Meraki Dashboard API to create a new network in PowerShell. In this post, we're going to perform both PUT & DELETE requests to update and then delete a resource - a network in our case.  As opposed to creating the script with the user inputs, as with my last two posts, we're going to skip that bit - but with the information from those two posts you should be able to add something in pretty simply.

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

Okie dokie. Let's get PowerShell ISE open and create the beginings of a script. In order to perform the PUT & DELETE requests, to modify and remove a resource on a server respectively, we are going to need two things - the ID of the organisation that the network belongs to, as well as the ID of that network. In the below text - which is largely formed of the script from the last post, and the one before that - we have defined a few variables and using Invoke-RestMethod we can see a list of organisations and their IDs in a response, then taking an ID we can perform a second request to get a list of networks and their IDs - 
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
$networks = Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/80009/networks" -Headers $Headers #80009 is an example
Out-Host -InputObject $networks
###
#Example output below (not script text)
###
id                   name          productTypes      
--                   ----          ------------      
L_1234567891         New Network 1 {switch, wireless}
N_1234567892         New Network 2 {switch}          
```
Select an organisation in which to create a network, note down the coresponding ID and check out the instructions written by Meraki [here](https://developer.cisco.com/meraki/api-v1/#!create-organization-network). The request is saying to send a POST request to /organizations/{organizationId}/networks - the same destination we previously sent a GET request to to retrieve the list of networks. A POST request is used to create data on a specific resource, like a web server (think a Facebook post). The instructions state that we will need to supply all data for the new network in JSON format - a common theme with Meraki. There is a sample body on that link, but for ease I've included it below - 
```json
{
    "name": "Long Island Office",
    "timeZone": "America/Los_Angeles",
    "tags": [ "tag1", "tag2" ],
    "productTypes": [
        "appliance",
        "switch",
        "camera"
    ]
}
```
... but let's take it back to basics and send only the required fields - name & product types. For the sake of this network, we'll create a combined network formed of wireless & switching. To do this, we will need to create the JSON body in PowerShell as the variable $body then convert it into the JSON format using the ConvertTo-Json cmdlet, stored as another variable called $jsonBody.
```powershell
$body = @{
    "name"              = "New Network"
    "type"              = "switch","wireless"
    }
$jsonBody = ConvertTo-Json -InputObject $body
```
For the request, it's as simple as the Invoke-RestMethod from the last post but with the -body switch followed by the variable with our JSON formatted body, swapping Get for Post against the -method switch and replacing {organizationId} with your organisation ID from a few steps back and you're good to go. Like so - 
```powershell
Invoke-RestMethod -Method Post -Uri "https://api.meraki.com/api/v1/organizations/{organizationId}/networks" -Headers $Headers -Body $jsonBody
```
Put together, with the response included, it'll look something like this - 
```powershell
$APIKey = "Enter your API key here"
$headers = @{
    "Content-Type" = "application/json"
    "X-Cisco-Meraki-API-Key" = $APIKey
}

$body = @{
    "name"              = "New Network"
    "productTypes"      = "switch","wireless"
    }
$jsonBody = ConvertTo-Json -InputObject $body

Invoke-RestMethod -Method Post -Uri "https://api.meraki.com/api/v1/organizations/{organizationId}/networks" -Headers $Headers -Body $jsonBody
Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/{organizationId}/networks" -Headers $Headers
###
#Example output below (not code text)
###
id               : L_Some-Network-ID
organizationId   : 1_Some-Organisation-ID
name             : New Network
productTypes     : {switch, wireless}
timeZone         : America/Los_Angeles
tags             : {}
enrollmentString : 
url              : https://n1.meraki.com/New-Network-swit/n/xxxxxxxx/manage/usage/list
notes            :
```
Whey! Network created. To make it a little more user friendly, having a person enter the product types & network name after typing the name of an organisation, you could end up with something like the below. The following example is, once saved, easy to run by right-clicking the file and selecting Run with PowerShell - 
```powershell
#----Headers
$APIKey = "Enter your API key here"
$headers = @{
    "Content-Type" = "application/json"
    "X-Cisco-Meraki-API-Key" = $APIKey
}
#----Get the list of organisations
$organisations = Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations" -Headers $Headers
#----Display the list of organisations
Out-Host -InputObject $organisations

#----Prompt the user to enter an organisation name and then save the ID of that organisation in a variable
$organisationName = read-host -Prompt "Please type an organisation name"
foreach ($organisation in $organisations){
    if ($organisation.name -like $organisationName.ToLower()) {
        $organisationId = $organisation.id
    }
}
#----Prompt the user to enter a network name and type(s) and store them as variables
$networkName = read-host -Prompt "Please type a new network name"
$networkType = (Read-Host "Please enter the type of network, it can be one or a combination (space separated) of the following - wireless | switch | appliance | systemsManager | camera | cellularGateway").ToLower().Replace("systemsmanager", "systemsManager").Replace("cellulargateway", "cellularGateway") -split " "
#----Create the body formed of the variables above
$body = @{
    "name"              = $networkName
    "productTypes"      = $networkType
    }
#----Convert the above variable to a new, JSON formatted variable
$jsonBody = ConvertTo-Json -InputObject $body

#----Create the network
Invoke-RestMethod -Method Post -Uri "https://api.meraki.com/api/v1/organizations/$($organisationId)/networks" -Headers $Headers -Body $jsonBody
#----Display the list of networks within that organisation, including the new one
Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/$($organisationId)/networks" -Headers $Headers
pause
```
We are using Read-Host to get the user to type in text, -split to split a string into an array, ConvertTo-Json to convert to JSON, .Replace to replace text and .ToLower to convert a string to lower case. Other than that, the script is very similar to the one in the last post. Again, the lines marked with a # are not processed by PowerShell and the ---- are used for ease of visibility. 

There we go. You can now script the creation of networks. There's not much different here to claiming devices, adding admin accounts, adding SSIDs, adding VLANs, etc., etc. The destination you'll need to add to the base URI for each query are found on [https://developer.cisco.com/meraki/api-v1](https://developer.cisco.com/meraki/api-v1). 

Happy scripting!
