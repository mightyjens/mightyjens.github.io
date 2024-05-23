---
title: 'Unifi settings to connect Home Assistant Bridge to Homekit'
date: 2024-05-23T09:30:01+02:00
#author: ["mightyjens"]
categories: ["linux"]
tags: ["Open Source","Selfhosted"]
categories_weight: 10
draft: false
comments: true
showReactions: 1
showToc: false
TocOpen: false
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
editPost:
    URL: "https://github.com/mightyjens/mightyjens.github.io/tree/main/content"
    Text: "Suggest Changes" 
    appendFilePath: true
---

That's a short one for Unifi (Network > 8.1.113) users struggling with the connection between the *Home Assistant Bridge* and *Apple Homekit*. In my installation, the connection attempt constantly ran into a timeout.

First of all make sure the port of the HASS Bridge is enabled in your firewall rules. In my case it's port **21064**, which should be the default one.

![HASS Bridge](../hass-homekit-bridge/2024-05-23_09-28.png)

In your **Network settings**, enable *Multicast DNS* and *IGMP Snooping* for the involved VLANs:

![Unifi Network](../hass-homekit-bridge/2024-05-23_09-25.png)

If you try to connect it via WiFi, enable the *Multicast Enhancement* in your **WiFi settings**:

![Unifi WiFi](../hass-homekit-bridge/2024-05-23_09-25_1.png)

Now *Homekit* should be able to connect to your *Home Assitant Bridge*. 