---
layout: post
title: "Integrating the Umbrella Enforcement API with Google Assistant"
author: "Jonny Winter"
categories: journal
tags: [Python,Flask,Azure,Umbrella,API,IFTTT,Google]
image: Creating-VLANs-CSV-Meraki-API-Flask.png
comments: true
---

*"The Cisco Umbrella Enforcement API is designed to give technology partners the ability to send security events from their platform/service/appliance within a mutual customer’s environment to the Umbrella cloud for enforcement. You may also list the domains and delete individual domains from the list. All received events will be segmented by the mutual customer and used for future enforcement."* - [Enforcement API on docs.umbrella.com](https://docs.umbrella.com/enforcement-api/docs)

## Summary

After stumbling onto Salnikov Andrey's [article](https://medium.com/@salnikov.andrei/hey-google-enable-my-meraki-guest-wifi-cc49bc01bc73) on Medium, I got inspired to do something similar but with a twist. I wanted to use Google Assistant & APIs, but I wanted to use the [Cisco Umbrella](https://umbrella.cisco.com/) [Enforcement API](https://docs.umbrella.com/enforcement-api/docs) to block a site on command rather than Meraki, use [Azure App Services](https://azure.microsoft.com/en-gb/services/app-service/) instead of [PythonAnywhere](https://www.pythonanywhere.com/) and feed a variable between [IFTTT](https://ifttt.com/) & [Flask](https://flask.palletsprojects.com/) as opposed to a static webhook endpoint. In this post I'm going to detail the steps required to simulate the below video in your own environment whilst adding in contextual information where relevant -

<a href="https://youtu.be/Bx4H-a5RQd8"><img alt="Umbrella API & Google Assistant" src="https://youtu.be/Bx4H-a5RQd8"/></a>

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

After that, click **continue** at the bottom and save your applet. At this point, you'll be able to give Google Assistant a command and you will see a POST message with the JSON data in the webhook.site page. Pretty neat! Try: "Hey Google, please block the site abc123.com". You will be able to see that domain referenced in the raw JSON. Our next step is to create the Python & Flask code in an Azure App Service so we can parse that JSON domain into a string which we provide in another JSON file up to Umbrella. It's easier than it sounds 😉.

## Azure




Happy scripting!
