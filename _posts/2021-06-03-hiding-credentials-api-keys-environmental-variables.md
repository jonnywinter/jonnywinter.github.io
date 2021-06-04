---
layout: post
title: "Hiding Authentication Information and Environmental Variables"
author: "Jonny Winter"
categories: journal
tags: [Python,Windows,API,macOS]
image: API-Keys-Environmental-Variables.png
comments: true
---

*"An environment variable is a variable whose value is set outside the program, typically through functionality built into the operating system or microservice. An environment variable is made up of a name/value pair, and any number may be created and available for reference at a point in time."* - [An Introduction to Environment Variables and How to Use Them by Jim Medlock on Medium](https://medium.com/chingu/an-introduction-to-environment-variables-and-how-to-use-them-f602f66d15fa)

## Summary

Working with credentials, wether username & password in [BASE64](https://www.base64decode.org/) or an API key, when programming causes us to face a problem - where to store them for my code to access? Writing them down inside your code, which may be copied/cloned/shared, is no way acceptable due to obvious risks. Of course, you could choose to enter them every time your code runs, which is secure to an extent but time consuming. What's the middle ground? In production environments, it's common to have a back end server that provides information to a front end server without exposing the API key to any third party. What about test/dev environments? In this post I'm going to detail a few options. 

## My Environment

Coffee: [House Roast, Original Blend from Union Hand-Roasted Coffee](https://unionroasted.com/products/house-roast-original-blend)
<br>
Music: [Funk & Soul Classics Playlist by Various](https://open.spotify.com/album/4FqQuWkP2W3bbc5ekIUmgH?si=CYoTp7l2Sjiwp25c7lirgQ)
<br>
OS: Windows 10 Pro v20H2 x64.
<br>
Python: [v3.9.2](https://www.python.org/downloads/release/python-392/)
<br>
IDE: [Visual Studio Code v1.56.2](https://code.visualstudio.com/)

## Tip o' the Hat

Rod's [answer](https://stackoverflow.com/questions/4906977/how-to-access-environment-variable-values) on Stack Overflow
<br>
Python OS [documentation](https://docs.python.org/3/library/os.html) on python.org
<br>
Malwarebytes blog [post](https://blog.malwarebytes.com/101/2017/01/explained-environmental-variables/) on Windows environmental variables
<br>
Raivat Shah's blog [post](https://medium.com/dataseries/hiding-secret-info-in-python-using-environment-variables-a2bab182eea) on Medium
<br>
Steve Griffith's [video](https://www.youtube.com/watch?v=3XjkaN8psp0&ab_channel=SteveGriffith-Prof3ssorSt3v3) on YouTube.

## Let's Begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**

For production, processes differ than in development - simply put, a front end server sends a request to a backend server, which in turn sends a request to a final destination server using an authentication medium that it has access to. Once the back end server receives data from the final destination server, it sends to the front end server. Essentially the front end server never has access to that authentication information and uses the back end server as a proxy. The back end server may store the authentication information in environmental variables, in a local config file or in some sort of secure repository/server that it has access to. This YouTube [video](https://www.youtube.com/watch?v=NpWWOS-tC5s&ab_channel=PortEXE) by PortEXE explains this process with [React](https://reactjs.org/).

In development, it really depends on your environment, if you're using shared source code, etc.

If the aim of the game is to simply want to store credentials outside of the source code, we can use something like an environmental variable. An environmental variable is a variable whose value is set outside of the program (like Python), typically through functionality built into the OS (like Windows). In this instance, I have a single Python file - app.py - which needs access to an API key. The following process explains how to store both temporary (cleared after the session/application is terminated) and persistent (remains after the session/application is terminated) on both Windows and macOS and access them in Python.

## Temporary Environmental Variables in Windows, macOS & Python

As mentioned, temporary environmental variables are only stored for the remainder of that session - looking at the following GIF of me displaying this in Command Prompt, if I were to close and re-open the application, I would not be able to re-call the variable. For me, in most applications these are next to useless - I've included them in this post to serve as contrast to persistent ones.

<a href="#"><img alt="Setting and showing environmental variables in command prompt" src="/assets/img/Temporary-Environmental-Variables-Command-Prompt.gif"/></a>

You can set and show envrionmental variables with the following syntax - 

Command Prompt
```cmd
::Set the environmental variables - 
set user=jonny
set pass=123456789

::Display the environmental variables - 
echo %user%'s password is %pass%
::Output = jonny's password is 123456789
```
PowerShell
```powershell
#Set the environmental variables - 
$env:user = "jonny"
$env:pass = 123456789

#Display the environmental variables - 
Write-Host $env:user"'"s password is $env:pass
#Output = jonny's password is 123456789
```
Terminal/Bash
```bash
#Set the environmental variables - 
export user="jonny"
export pass="123456789"

#Display the environmental variables - 
echo $user"'"s password is $pass
#Output = jonny's password is 123456789
```
Python (for completeness, but missing the point)
```python
import os

#Set the environmental variables - 
os.environ['user'] = "jonny"
os.environ['pass'] = "123456789"

#Display the environmental variables -
print(os.environ['user'] + "'s password is " + os.environ['pass'])
#Output = jonny's password is 123456789
```

## Persistent Environmental Variables in Windows & macOS and Retrieving Them in Python

This [Help Desk Geek blog post](https://helpdeskgeek.com/how-to/create-custom-environment-variables-in-windows/) written by Aseem Kishore goes into creating this in depth, but from a high level - 
1. Open **System Properties** by going to **Settings** > **System** > **About** and clicking **Advanced system settings**.
2. On the **Advanced** tab, click **Environmental Variables...**.
3. Here, click **New** under **User variables for %username%**. 
4. Give the variable a name, for example *user* followed by a value, like *jonnywinter*
5. Click **OK** to save.

<a href="#"><img alt="Setting an environmental variable in Windows" src="https://helpdeskgeek.com/wp-content/pictures/2012/09/environment-variables-dialog.png"/></a>

That's really it for Windows. You can now open up either Command Prompt or PowerShell and display the list of variables or that specific variable by - 

Command Prompt
```cmd
::Display the list of environmental variables
set
::Display a specific environmental variable
echo %user%
```
PowerShell
```powershell
#Display the list of environmental variables
Get-ChildItem -path env:
#Display a specific environmental variable
$env:user
```
For macOS, the process is a little different; the environmental variables are stored in a file that we must edit in a text editor (and if using nano press **CTRL+W** to quit, selecting **Y** to save). The easiset way to do this is to open up Terminal and - 

Terminal/Bash
```bash
#Navigate to your home directory
cd ~
#Open up the hidden file, .bash_profile (or .zprofile if your using ZSH and not Bash)
nano .bash_profile
```
Nano
```bash
export user="jonny"
export pass="123456789"
```
Once saved, the variables will be persistent if you close & re-open Terminal. Like with the above, you can call the variables in Terminal by using *echo*. With the above steps complete, we can now go about calling those variables from within Python. It's the same process for macOS or Windows.

Python
```python
import os

user = os.environ.get("user")
pass = os.environ.get("pass")

print(user + "'s password is " + pass)
#Output = jonny's password is 123456789
```
In the above code, you are importing the *os* module and using the *environ.get* method to retrieve the environmental variables, setting them as python variables *user* and *pass* respectively.   

## Storing Authentication Information Outside of Source Code in JSON

As an alternative to environmental variables, storing something like an API key in a file in a local directory or secure server can be a great option. This could easily be YAML or XML, but I like working with JSON so I'm using that in this example. The directory could be an absolute path (i.e. c:\foo\bar.json) or simply relying on the working directory (i.e. bar.json). The great thing here is that your source code, once again, doesn't have your authentication information within it but instead references another place that does - 

Python
```python
import os
import json

config = json.load(open('C:\\app\\config.json', 'r'))

print(config['apikey'])
#Output = 123-456-789
```
JSON
```json
{
  "apikey":"123-456-789"
}
```
## Using GitHub & .gitignore

GitHub is fantastic, and using it is great. However, if you start to commit directories & files to the cloud and share them with others then you're going to need to exclude files & folders from being synced. Luckily the creators of Git thought of this and came up with the [.gitignore file](https://git-scm.com/docs/gitignore). Although the .gitignore file is synced to the cloud, the files and folders that the file references will not be. This YouTube [video](https://www.youtube.com/watch?v=17UVejOw3zA&t=334s&ab_channel=TheCodingTrainTheCodingTrain) by The Coding Train (~06:10 mins in) goes into creating this file and specifically not syncing a .env file which contains an API key. To do this - 
1. The .gitignore file must exist in the root directory and will be created manually.
2. You can specify files (i.e. config.json) & folders (i.e. project_notes/).
3. You can use the astrix wildcard to specifiy 


Happy scripting!
