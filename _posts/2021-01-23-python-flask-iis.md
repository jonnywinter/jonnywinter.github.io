---
layout: post
title: "Deploying a Python Flask app in IIS"
author: "Jonny Winter"
categories: journal
tags: [IIS,Python,Flask]
image: Python-Flask.png
---

*"Flask is a lightweight WSGI web application framework. It is designed to make getting started quick and easy, with the ability to scale up to complex applications. It began as a simple wrapper around Werkzeug and Jinja and has become one of the most popular Python web application frameworks."* - [Armin Ronacher on The Pallets Projects](https://palletsprojects.com/p/flask/)
(*)
## Summary

Python has quickly become a very, if not the most, popular programming language, and Flask is a framework that can be used to run web apps within it. Flask can scale from small apps to much larger ones, but when compared with other alternatives, like Django, it is suited to smaller applications. For me, Flask is simple, easy to use and quick to troubleshoot, so I think of it when I think of web apps. Although Flask has the ability to run a web server, it's not designed for production purposes like IIS or Apache are - as I'm an avid Windows user I'm sticking with IIS. With the addition of WSGI (**W**eb, **S**erver, **G**ateway & **I**nterface) we can pass requests from IIS to Flask. In this post I'm going to show the steps required to get IIS to serve a Flask app, display a .html web page and pass data between them. 

## Requirements

Other than the free software, you will need either a copy of Windows Server or install the IIS features in Windows (10) Pro. If you are to perform the bits on a Windows Desktop OS, then you'll be able to install IIS by -
<br>
#1 - Open up the Programs and Features control panel applet (or use run dialog + appwiz.cpl).
<br>
#2 - Click Turn Windows features on or off.
<br>
#3 - Locate Internet Information Services
<br>
<br>
<a href="#"><img alt="Installing IIS on Windows Desktop OS" src="/assets/img/IIS-Windows-Desktop.png"/></a>

## My Environment

Coffee: [Timana, Colombia from Union Hand-Roasted Coffee](https://unionroasted.com/products/timana-colombia)
<br>
Music: [Hello Nasty by Beastie Boys](https://open.spotify.com/album/2cT6Yb6EWcBhyqGd7DXeL2?si=EDW8IUMWR7mOpHTpCBk23g)
<br>
OS: Windows Server 2012 Datacenter (but also tested on Windows 10 Pro v20H2 x64). 
<br>
Python: [Python 3.9.0](https://www.python.org/downloads/release/python-390/)
<br>
Packages: [Flask v1.1.2](https://pypi.org/project/Flask/)
<br>
IDE: [Visual Studio Code v1.52.1](https://code.visualstudio.com/)

## Tip o' the Hat

The [Flask documentation](https://flask.palletsprojects.com/en/1.1.x/) by Pallets
<br>
Bilal Bayasut's [post](https://medium.com/@bilalbayasut/deploying-python-web-app-flask-in-windows-server-iis-using-fastcgi-6c1873ae0ad8)  on Medium
<br>
The [WFastCGI documentation](https://pypi.org/project/wfastcgi/) by Microsoft
<br>
This [Stack Overflow post answere](https://stackoverflow.com/questions/50592847/web-platform-installer-python-installer-downloaded-file-failed-signature-veri) by William Sousa
<br>
KiranH's [post and document](https://techcommunity.microsoft.com/t5/iis-support-blog/how-to-run-python-application-on-iis-that-uses-flask-framework/ba-p/812898) in the Microsoft Tech forum
<br>
Michael Fore's [video](https://www.youtube.com/watch?v=En9vo7Ognm0&t) on YouTube (and [GitHub](https://github.com/Michael-fore))

## Let's begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**

Now you've generated your API key and created a dashboard account (if you do not already have these), open PowerShell ISE so we can go about calling the Dashboard [REST API](https://www.youtube.com/watch?v=7YcW25PHnAA&t=1s&ab_channel=WebConcepts). To get started, we are going to perform a [GET request](https://www.youtube.com/watch?v=guYMSP7JVTA&ab_channel=Telusko) to fetch all of the organisations in which your dashboard administrator account has access to - if you have access to the [MSP Portal](https://documentation.meraki.com/General_Administration/Organizations_and_Networks/Using_the_MSP_Portal_to_Manage_Multiple_Organizations) you will return multiple organisations, if not then you will only return one. A GET request is used to retrieve data from a specific resource, like a web server.

In PowerShell, we're going to be using the [Invoke-RestMethod](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-restmethod?view=powershell-7.1) cmdlet to interact with the Meraki Dashboard REST API and storing the returned values in a [variable](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_variables?view=powershell-5.1). Once these variables have a value we will then display the values in a nicely formatted table.

Firstly, create a new script (File > Save) and type the following text to store your API key in the $APIKey variable.
```powershell
$APIKey = "Enter your API key here"
```
**&lt;NOTE>**: Storing an API key, as with any credentials, in clear text is a security risk - your API key is effectively your username & password. If a malicious party were to obtain this key, they would be able to authenticate as you. There are many well documented ways on how to secure this in a production environment, i.e. [here](https://www.freecodecamp.org/news/how-to-securely-store-api-keys-4ff3ea19ebda/). This is a purely educational project, do this at your own risk. **&lt;/NOTE>**.

When we communicate with the Meraki Dashboard API, the server requires a few headers are passed along with the URI each request; X-Cisco-Meraki-API-Key, your API key, & Content-Type. The Content-Type header is provided to let the server know that any data sent to the server is provided in JSON (**J**ava**S**cript **O**bject **N**otation). In PowerShell, we'll include both of these in a single variable called $headers - the @ is used when we put data on multiple lines. 
```powershell
$headers = @{
    "X-Cisco-Meraki-API-Key" = $APIKey
    "Content-Type" = "application/json"
}
```
Now we've got our API key & headers defined as variables, we can put them to use in a GET request using Invoke-RestMethod. The switches for the cmdlet are self explanitory - -Uri is the URI, -Headers is the headers & -Method is the CRUD method. Easy mode. The URI we are using is obtained from the [Meraki Dashboard API docs](https://developer.cisco.com/meraki/api-v1/#!get-organizations), all we need to do is prepend each request with https://api.meraki.com/api/v1.
```powershell
Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations" -Headers $Headers
```
You should receive a response something like this - 
<br>
<br>
<a href="#"><img alt="Meraki API PowerShell Get Organisations" src="/assets/img/Meraki-API-PowerShell-Get-Organisations.png"/></a>
<br>
<br>
What you can see is the organisation name (in the pink rectangle) and the organisation ID (in the green rectangle - unique to the organisation). Using the organisation ID and the same headers as before (included here for completeness), we can do things like displaying the number of [networks within the organisation](https://developer.cisco.com/meraki/api-v1/#!get-organization-networks) by simply replacing {organizationId} with the organisation ID against the name of the organisation you want to query - 
```powershell
$APIKey = "Enter your API key here"
$headers = @{
    "Content-Type" = "application/json"
    "X-Cisco-Meraki-API-Key" = $APIKey
}
Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/{organizationId}/networks" -Headers $Headers
```
You should receive a response something like this - 
```powershell
id               : L_Network-ID-1
organizationId   : XXXXX
name             : Network Name 1
productTypes     : {camera, switch, wireless}
timeZone         : Europe/London
tags             : {}
enrollmentString : 
url              : https://nXXX.meraki.com/XXXXXXXXXXX/n/XXXXXXXXX/manage/usage/list
notes            : 

id               : L_Network-ID-2
organizationId   : XXXXX
name             : Network Name 2
productTypes     : {appliance, switch, wireless}
timeZone         : America/Los_Angeles
tags             : {}
enrollmentString : 
url              : https://nXXX.meraki.com/XXXXXXXXXXXXX/n/_XXXXXXXXX/manage/usage/list
notes            : 
```
**&lt;NOTE>**: Most objects have an ID, this is aparent with the network ID above. Devices use their serial number instead, but unique IDs are common. **&lt;/NOTE>**.

Great! Only thing is, it looks kind-a iffy. So, let's display it in a table and only display the network ID, organization ID, network name & product types. To do this, put the Invoke-RestMethod request in a variable. Then, call that variable followed by Format-Table cmdlet, like so - 
```powershell
$request = Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/{organizationId}/networks" -Headers $Headers
$request | Format-Table id,organizationId,name,productTypes

###
#Example output below (not script text)
###
id                   organizationId name        productTypes                 
--                   -------------- ----        ------------                 
L_Network-ID-1       XXXXXX         Network 1   {camera, switch, wireless}   
L_Network-ID-2       XXXXXX         Network 2   {appliance, switch, wireless}
```
Using a little ingenuity you can make these steps automatic by simply typing in the name of the organisation, finding that name using the if cmdlet and storing the ID relating to it in a variable. You can then place this variable in the Invoke-RestMethod -Uri string and query networks without ever having to type in an ID. The following example is, once saved, easy to run by right-clicking the file and selecting Run with PowerShell.
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
ForEach ($organisation in $organisations){
    if ($organisation.name -like $organisationName.ToLower()) {
        $organisationId = $organisation.id
    }
}
#----Using that organisation ID, display the list of networks in a formatted table
$networks = Invoke-RestMethod -Method Get -Uri "https://api.meraki.com/api/v1/organizations/$($organisationId)/networks" -Headers $Headers
$networks | Sort-Object Name | Format-Table Name,ID,productTypes
pause
```
The for loop is a common thing in scripting. In PowerShell the syntax is ForEach and well documented [here](https://devblogs.microsoft.com/scripting/basics-of-powershell-looping-foreach/) by Dr Scripto. The lines marked with a # are not processed by PowerShell and the ---- are used for ease of visibility. 
<br>
<br>
But yeah, there we go. You can now query all of your networks over all of your organisations via PowerShell using the Meraki Dashboard API. From here, using this/similar method, you can query things like MS switches, switch ports, MR Wi-Fi APs, SSIDs, etc., etc. The destination you'll need to add to the base URI for each query are found on [https://developer.cisco.com/meraki/api-v1](https://developer.cisco.com/meraki/api-v1). 

Happy scripting!
