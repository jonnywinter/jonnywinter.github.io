---
layout: post
title: "Integrating the Umbrella Enforcement API with Google Assistant"
author: "Jonny Winter"
categories: journal
tags: [Python,Flask,Azure,Umbrella,API,IFTTT,Google]
image: Umbrella-Google-Assistant-Thumbnail.png
comments: true
---

*"The Cisco Umbrella Enforcement API is designed to give technology partners the ability to send security events from their platform/service/appliance within a mutual customer’s environment to the Umbrella cloud for enforcement. You may also list the domains and delete individual domains from the list. All received events will be segmented by the mutual customer and used for future enforcement."* - [Enforcement API on docs.umbrella.com](https://docs.umbrella.com/enforcement-api/docs)

## Summary

After stumbling onto Salnikov Andrey's [article](https://medium.com/@salnikov.andrei/hey-google-enable-my-meraki-guest-wifi-cc49bc01bc73) on Medium, I got inspired to do something similar but with a twist. I wanted to use Google Assistant & APIs, but I wanted to use the [Cisco Umbrella](https://umbrella.cisco.com/) [Enforcement API](https://docs.umbrella.com/enforcement-api/docs) to block a site on command rather than Meraki, use [Azure App Services](https://azure.microsoft.com/en-gb/services/app-service/) instead of [PythonAnywhere](https://www.pythonanywhere.com/) and feed a variable between [IFTTT](https://ifttt.com/) & [Flask](https://flask.palletsprojects.com/) as opposed to a static webhook endpoint. In this post I'm going to detail the steps required to simulate the below video in your own environment whilst adding in contextual information where relevant -

<iframe width="560" height="315" src="https://www.youtube.com/embed/Bx4H-a5RQd8" frameborder="0" allowfullscreen></iframe>


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

## Prerequisites

1. A Cisco Umbrella subscription (trial or full).
2. An Azure subscription (trial or full), but the App Service is free for the first 10. See F1 [here](https://azure.microsoft.com/en-gb/pricing/details/app-service/windows/).
3. An IFTTT subscription (free or paid). The standard subscription is free for 3 integrations, but you will need to re-activate your integration once every 3 months to keep it from being de-activated. Some handy information [here](https://help.ifttt.com/hc/en-us/articles/360053706813-IFTTT-Plans-at-a-glance).
4. Access to a Google Assistant device. I'm using a Sonons product here, but it also worked with my Google Pixel 4a handset.

## Let's Begin

**&lt;NOTE>**: Instead of re-inventing the wheel and explaining things that have been well defined by someone else, I have included links next to some words/technologies/acronyms/protocols that I feel could proove useful to those not yet 'in the know'. **&lt;/NOTE>**

Unfortuanately we're going to need to dip in and out of the 1, 2 & 3 of the prerequisites (i.e. we can't do one at a time) so take note of the headings for each section change.

## Umbrella

Having already created your account, make your way to the [dashboard](https://dashboard.umbrella.com). On the left side, navigate to **Policies** > **Policy Components** > **Integration** and click add to create a new integration. Give your integration a name, for this example I'm going to call it *Umbrella Google Assistant*. Once you've created it, **save** it and open it to **enable** it. Here, make note of the **Custom Domains** list; this is what we're going to be adding domains to via our voice commands. Before exiting this bit, copy the **API URL**.

**&lt;NOTE>**: The API URL is formed of *https://s-platform.api.opendns.com/1.0*, */events* and the query string *?&customerKey=1111-2222-3333-4444*. The first part is common amongst all Enforcement API endpoints, the second is specifically where we will send POST requests to add domains to our block list & the last bit is your integration specific API key - keep this safe as it is the only way that your request is validated. **&lt;/NOTE>**

Next we need to our integration to a policy that affects one/more devices which will be affected by this project. To do this, navigate to **Policies** > **DNS Policies** > *Select policiy*. Here, click **Security Setting Applied** to edit your policy, **Categories to Block** and then edit. **Enable** your newly created integration, **save** > **proceed** > **set & return** > **save**. We're now done with the Umbrella part.

**&lt;NOTE>**: Within **Reporting** > **Management** > **Admin Audit Log** > **Filter** > **Filter by Identities & Settings: Umbrella Google Assistant Domain List** you can see/report on when domains are added to your domain list, as well as in the **Custom Domains** section of the integration. **&lt;/NOTE>**

## IFTTT

IFTTT is the platform that we will use to integrate our Azure webhook endpoint to our Google Assistant commands. Head to their site and create an account - for this post I created a Standard account, which is free. Once you've signed up and signed in, locate the ability to **Create** a new integration. They come in two components, 'if this' and 'then that' - 

### If This
**Add** > select **Gooogle Assistant** > select **Say a phrase with a text ingredient** > *connect your Google Assistant account, if not already done*. In the next screen you will select what you want to say (the $ symbols mean variable) -
1. **What do you want to say?**: Block $
2. **Alternate**: Block the site $
3. **Alternate**: Please block $
4. **Response**: Sure, blocking $

Before we get to the *then that* section, head over to webhook.site and create yourself a test webhook endpoint (it just appears in the middle of the screen). Copy the **Your unique URL** url. We will use this to check that Google Assistant is sending POST messages.

### Then That: 
**Add** > select **Webhook** > select **Make a web request**. In the next screen you will dictate what Google Assistant will send you and where - 
1. **URL**: *your unique URL from webhook.site*
2. **Method**: POST
3. **Content Type**: application/json
4. **Body**: *{"data":"{{TextField}}"}*

**&lt;NOTE>**: The text in step 4 above is JSON; the key is *data* and the value is the *TextField* variable. This forms a key-value pair. **&lt;/NOTE>**

After that, click **continue** at the bottom and save your applet. At this point, you'll be able to give Google Assistant a command and you will see a POST message with the JSON data in the webhook.site page. Pretty neat! Try: "Hey Google, please block the site abc123.com". You will be able to see that domain referenced in the raw JSON. Our next step is to create the Python & Flask code in an Azure App Service so we can parse that JSON domain into a string which we provide in another JSON file up to Umbrella. It's easier than it sounds 😉.

We will need to return to IFTTT to change the webhook.site endpoint to be the URL given to us by Azure in the next step.

## Azure

You are given 10 free Azure App Services at all time, which is great. Using one of these free F1 App Services & VScode we can have this last step configured in no time. So, head over to [Azure](https://portal.azure.com) and locate and open **Free Services**. Under **Always free services**, select **App Service** and then click **Create**. Here, create a new **Resource group** and give it a name, like *Umbrella Google Asssistant Tie-in* or something memorable. In the **Instance Details** screen - 
1. **Name**: *umbrellagoogleassistanttiein* or something like that
2. **Publish**: Code
3. **Runtime Stack**: Python 3.8
4. **OS**: Linux
5. **Region**: I did mine in UK South.

The **App Service Plan** is the next bit to fill in, here the only thing you need to change/select is **Free F1** under **Dev/Testing** in the **SKU & Size** setting. Lastly, **Review & Create**. Once it's created (will take a few mins), go ahead and open up your newly created resource and copy the **URL**. 

At this point, paste the newly copied URL into IFTTT replacing the webook.site endpoint. That's it for IFTTT. 

On your computer, open up [VScode](https://code.visualstudio.com/) and install the [Azure App Services add-in](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice). Once installed, sign-in and open it up. Within a few clicks you'll see your App Service under your subscription. At this point, create a new folder on your computer and point VScode to it. Create two files in the root of the directory (see below). 
1. The requirements text file is used to tell Azure which Python packages we require in our app.
2. The only thing to change in the Python file is your customer key, whcih can be found in the first of the two Umbrella notes earlier in the post.

requirements.txt -
```text
Flask>=1.0,<=1.1.2
requests
```
app.py - 
```python
import json
import requests
from datetime import datetime
from flask import Flask, request

customerKey = 'ENTER YOUR CUSTOMER KEY HERE'
postUrl = f'https://s-platform.api.opendns.com/1.0/events?customerKey={customerKey}'
getUrl = f'https://s-platform.api.opendns.com/1.0/domains?customerKey={customerKey}'

headers = {
    'Content-Type':'application/json',
    'Accept': 'application/json'
}

app = Flask(__name__)

#Uncomment the following three lines if you want to display a page so you can check to see easily if your app is up and working
#@app.route("/")
#def hello():
#    return "Latest udate: 1.1"

@app.route('/block',methods=["POST"])
def block():

    domainJson = request.get_json()
    domain = (domainJson['data']).replace(" ", "")

    print(domain)
    now = str(datetime.now().isoformat()) + 'Z'

    data = {
        "alertTime": now,
        "deviceId": "ba6a59f4-e692-4724-ba36-c28132c761de",
        "deviceVersion": "13.7a",
        "dstDomain": domain,
        "dstUrl": "http://" + domain + "/",
        "eventTime": now,
        "protocolVersion": "1.0a",
        "providerName": "Security Platform"
    }

    #Uncomment out the following two GET lines if you wan to see domains in your block list
    post = requests.request('POST', postUrl, headers=headers, data=json.dumps(data)).json()
    #get = requests.request('GET', getUrl, headers=headers).json()
    print(post)
    #print(get)

    return 'success', 202
```
Noteworthy code - 
1. The data JSON body is standardised and all feilds are required (most of which mean little-to-nothing to me) as documented [here](https://docs.umbrella.com/enforcement-api/docs/events2). 
2. The line *domain = (domainJson['data']).replace(" ", "")* is used to remove any spaes created by Google Assistant, which it adds every time.
3. The datetime specified in the JSON data body has to be provided in ISO standard and appended with a Z at the end, which is the zone designator for zero UTC offset.
4. If you want to know more about the above code, my most recent three posts have gone into Flask in more detail. They should help out here, or the [official documentation](https://palletsprojects.com/p/flask/).

**&lt;NOTE>**: The code isn't included above for brevity, however you could run a Flask web server and use Postman to send POST requests with the same JSON body you saw from webhook.site to see how your code parses the data and if it errors out - the above code works, but if you change anything this method could really help! **&lt;/NOTE>**

Once you've copied & pasted in the code, right click your App Service in the Azure extension and select **Deploy to web app**. After a few minutes you will see a success message presented to you from VScode, then your app is good to go!

Although we're using the Umbrella Enforcement API, Salnikov Andrey used the Meraki API to disable/enable an SSID which goes to show a common use case for this. With minor changes, this can be used to embed data into any JSON body for any POST/PUT/DELETE request. Time to get creative! :D 

Happy scripting!
