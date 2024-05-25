---
title: 'Ferraris electricity meter going smart using ESP32 Cam and AI on the Edge'
date: 2024-05-25T09:41:45+02:00
draft: true
categories: ["home assistant"]
tags: ["Open Source","Selfhosted","Home Assistant"]
series: [""]
categories_weight: 30
comments: true
showReactions: 1
showToc: true
TocOpen: false
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
UseHugoToc: true
---

Here is a small summary from various sources on how to make an old Ferraris electricity meter smart using the ESP32 + CAM module. The aim of the exercise will be to make the data visible in the Home Assistant energy dashboard and thus determine total consumption.

## The Equipment  

- <a href="https://amzn.to/3VcMk7J" target="_blank">ESP32 + CAM Module</a>
- <a href="https://amzn.to/4dUeTho" target="_blank">Micro SD Card</a>
- <a href="https://www.thingiverse.com/thing:5567461" target="_blank">3D-printed Power Meter Holder</a>
- 80mm x 30mm acrylic glass mirror

### Prepare the Lens

Since the lens has a fixed focus that is glued, we need to make a small change to the hardware. We need to remove this small shiny edge on the lens so that we can adjust the focus. This way we could manually madify the focus to achieve the optimum image quality. 

## Setup

### Initial Setup

As we are setting the ESP32 up using the <a href="https://github.com/jomjol/AI-on-the-edge-device" target="_blank">AI on the Edge</a> project, the initial setup is straight forward. 

1. Format your SD card using FAT
2. Connect the device in Bootloader mode (FLASH button pressed while the RESET button gets relesed) to your pc using the USB interface
3. Navigate to the <a href="https://jomjol.github.io/AI-on-the-edge-device/index.html" target="_blank">Webinstaller and Console</a> and connect the device
4. After initially flashing the ESP32 firmware, the SD card can be set up automatically, otherwise have a look at the <a href="https://jomjol.github.io/AI-on-the-edge-device-docs/Installation/#3-sd-card" target="_blank">documentation</a>

All further updates could then be installed over-the-air.

### Setup WiFi

### Setup the AI


### Connect MQTT Broker


### Add to Home Assistant



## Verdict