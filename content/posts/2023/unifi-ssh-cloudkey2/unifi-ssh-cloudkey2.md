---
title: 'SSH into your Unifi CloudKey Gen2'
date: 2023-06-19T09:43:30+02:00
draft: false
author: ["mightyjens"]
categories: ["unifi"]
tags: ["Unifi","IT"]
series: [""]
categories_weight: 10
comments: true
showToc: false
TocOpen: false
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
UseHugoToc: true
---
For anyone who struggles to get into their Ubiquity CloudKey Gen2 via SSH… There are a bunch of forum posts swarming around offering a wide variety of options. ubnt/ubnt, root/ubnt… For me, using version “UniFi OS UCK G2 Plus 3.1.9” the following combination worked:

Setup the SSH password in the CloudKey:

![Thule EasyFold XT 2](../image-1024x357.webp)

Now you could use the username root and the password you just setup to SSH into your CloudKey:

![Thule EasyFold XT 2](../image-1.webp)