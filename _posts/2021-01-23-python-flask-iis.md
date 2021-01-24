---
layout: post
title: "Deploying a Python Flask app in IIS"
author: "Jonny Winter"
categories: journal
tags: [IIS,Python,Flask]
image: Python-Flask.png
---

*"Flask is a lightweight WSGI web application framework. It is designed to make getting started quick and easy, with the ability to scale up to complex applications. It began as a simple wrapper around Werkzeug and Jinja and has become one of the most popular Python web application frameworks."* - [Armin Ronacher on The Pallets Projects](https://palletsprojects.com/p/flask/)

## Summary

Python has quickly become a very, if not the most, popular programming language, and Flask is a framework that can be used to run web apps within it. Flask can scale from small apps to much larger ones, but when compared with other alternatives, like Django, it is suited to smaller applications. For me, Flask is simple, easy to use and quick to troubleshoot, so I think of it when I think of web apps. Although Flask has the ability to run a web server, it's not designed for production purposes like IIS or Apache are - as I'm an avid Windows user I'm sticking with IIS. With the addition of WSGI (**W**eb, **S**erver, **G**ateway & **I**nterface) we can pass requests from IIS to Flask. In this post I'm going to show the steps required to get IIS to serve a Flask app, display a .html web page and pass data between them. 

## Requirements

Other than the free software, you will need a copy of Windows Server. You can install the IIS features in Windows (10) Pro to bypass this, but you're not realistically going to use a desktop OS as your web server. However, if you wanted to perform the bits on a Windows Desktop OS, you'll be able to install IIS by -
<br>
1. Open up the Programs and Features control panel applet (or use run dialog + appwiz.cpl).
<br>
2. Click Turn Windows features on or off.
<br>
3. Locate Internet Information Services.
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

Get logged into the server you wish to host your web app on and follow the numbered steps below (text borrowed from the [IONOS by 1&1 help page](https://www.ionos.co.uk/help/server-cloud-infrastructure/server-administration/installing-iis-on-a-server-with-windows-server-minimal-installation/#c101715)) to get IIS installed, or run the PowerShell command below - 
```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
Install-WindowsFeature -name Web-CGI
#Or
Install-WindowsFeature -Name Web-Server -IncludeAllSubFeature
```
1. Press the **Windows** key and select **Server Manager**.
2. In the **Server Manager Dashboard**, click **Manage** > **Add Roles and Features**.
3. Click **Installation Type**.
4. Select the **Role-based or feature-based installation** option and click **Next**.
5. Select the server on which you want to install IIS and click **Next**.
6. Activate the **Web Server (IIS)** role.
7. To add the IIS Management Console, click **Add Features**.
8. Click **Next**. The **Select Features** window will open.
9. Click **Next**. The Web Server Role (IIS) window will open.
10. Click **Next**. The Select **Role Services** window will open.
11. Select **CGI** under **Application Development Features** and click **Next**.
12. To install the selected roles, role services, and features, click **Install**.
13. To complete the installation, click **Close**.


Happy scripting!
