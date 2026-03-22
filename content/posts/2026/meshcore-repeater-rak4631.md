---
#slug: title
title: 'Meshcore: Repeater based on RAK4631'
date: 2026-03-22T09:04:01+02:00
draft: false
#author: ["mightyjens"]
categories: ["meshcore"]
tags: ["Meshcore", "LoRa"]
series: [""]
categories_weight: 10
#cover:
#  image: posts/welcome-new-blog/msg.png
#  caption: ""
#  hiddenInList: true
comments: true
showReactions: 1
showToc: true
TocOpen: false
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
UseHugoToc: true
#editPost:
#    URL: "https://github.com/mightyjens/mightyjens.github.io/tree/main/content"
#    Text: "Suggest Changes" 
#    appendFilePath: true  
---
## Meshcore Repeater: DE-KO j3nstastic

This is my home repeater for *Meshcore*.

### Core Components

| Component | Specs | Function |
|-----------|-------|----------|
| **RAK19007** | Dual carrier board | Modular base for WisBlock sensors |
| **RAK4631** | nRF52840 + SX1262 LoRa | 868 MHz core, Bluetooth LE, ARM Cortex-M4 |
| **18650 Battery** | 3.7V 3000mAh Li-Ion | > 48h autonomy (no sun) |
| **Solar Panel** | 6V/3W polycrystalline | ~1.2A peak charging |
| **MPPT Controller** | 1A TP4056-based | Max Power Point Tracking, overcharge protection |

### Technical Highlights
| Spec | Value |
|-----|-------|
| **Frequency** | 868 MHz EU ISM |
| **TX Power** | 20 dBm |
| **Antenna Gain** | 5.5 dBi |
| **Urban Range** | 1.2 km |
| **Rural Range** | 8+ km LOS |
| **TX Current** | 120 mA |
| **RX Current** | 25 mA |
| **Sleep Current** | 2 µA |


**Key Features:**
- **RAK WisBlock ecosystem** - Stackable modules for future expansion
- **Intelligent power management** - Solar-first, battery backup
- **N-Male 5.5dBi antenna** - Optimized for EU 868 MHz propagation
- **3D-printed enclosure** - PETG UV-resistant, fully weatherproof


![RAK4631 Repeater internals](../meshcore-repeater-rak4631/RAK4631_3.png)
![Solar-powered enclosure](../meshcore-repeater-rak4631/RAK4631_2.png)
![Complete assembly with antenna](../meshcore-repeater-rak4631/RAK4631_1.png)