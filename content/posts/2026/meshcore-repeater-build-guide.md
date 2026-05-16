---
#slug: meshcore-repeater-build-guide
title: 'Building a Solar-Powered MeshCore Repeater with RAK WisBlock'
date: 2026-05-16T12:00:00+02:00
draft: false
#author: ["mightyjens"]
categories: ["meshcore"]
tags: ["Meshcore", "LoRa"]
series: [""]
categories_weight: 10
#cover:
#  image: posts/meshcore-repeater-build-guide/cover.png
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

## Overview

This post documents the hardware build of my solar-powered standalone [MeshCore](https://meshcore.co.uk/) repeater. The design prioritises low power consumption, clean RF performance, and weather resistance — running completely unattended on a 10 W solar panel and a single protected 18650 cell.

## Credits

A big thank you to a fellow member of the **MeshCore Rheinland** group, who came up with the original idea and pointed me towards exactly the right components. Without that nudge this build would still be on a whiteboard.

---

## Hardware

### Mainboard: RAK 19007 + RAK 4631

The core of the build is a two-piece WisBlock stack: the **RAK 19007** base board and the **RAK 4631** core module.

#### RAK 19007 – WisBlock Base Board

The RAK 19007 is a compact (30 × 60 mm) WisBlock carrier that takes care of power management so the core module doesn't have to:

- Integrated LiPo/Li-Ion charge controller — no external TP4056 module needed
- **Solar input** via JST 1.25mm 2-pin connector wired directly to the charge controller, bypassing the USB negotiation layer entirely
- USB-C port for programming and debugging
- WisBlock slot system: expandable with GPS, additional sensors, or a second radio

#### RAK 4631 – WisBlock Core Module

The RAK 4631 combines a Nordic nRF52840 microcontroller with a Semtech SX1262 LoRa transceiver on a single stamp-sized module:

| Spec | Value |
|------|-------|
| MCU | Nordic nRF52840 (ARM Cortex-M4 @ 64 MHz) |
| Flash / RAM | 1 MB / 256 KB |
| LoRa chip | Semtech SX1262 |
| Frequency | 868 MHz (EU868) |
| TX Power | up to +22 dBm |
| RX Sensitivity | down to −148 dBm |
| RF connector | U.FL (internal) |
| Sleep current | ~3–5 µA (Deep Sleep) |
| Wireless | BLE 5.0 (for config via app) |

The deep-sleep current of just 3–5 µA is a meaningful advantage over ESP32-based LoRa boards, which typically idle at several hundred µA. On a small solar cell and a single 18650, that difference adds up quickly overnight.

![Open enclosure showing the RAK 19007 + RAK 4631 WisBlock stack mounted on the right, with the 18650 battery holder on the left and the N-Female antenna bulkhead at the top](/posts/2026/Building-a-Solar-Powered-MeshCore-Repeater-with-RAK-WisBlock/IMG_0027.JPEG)

---

### Battery: Keeppower 18650 – 3500 mAh

| Spec | Value |
|------|-------|
| Chemistry | Li-Ion |
| Nominal voltage | 3.6 V (max 4.2 V) |
| Capacity | 3500 mAh |
| Protection (PCM) | Over-charge, deep-discharge, short-circuit |

The RAK 19007 charges at roughly 500 mA, well below 1C for this cell — so the PCM protection circuit won't trigger during normal solar charging. The built-in protection makes it suitable for unattended, long-term outdoor operation.

---

### Antenna: ALFA LoRa 868 MHz 5 dBi

| Spec | Value |
|------|-------|
| Manufacturer | ALFA Networks |
| Frequency | 868 MHz (EU ISM) |
| Gain | 5 dBi (omnidirectional, vertically polarised) |
| Vertical beamwidth | ~25–30° |
| Form factor | Outdoor-rated weatherproof stick |
| Connector | N-Male |
| Source | [getic.de](https://www.getic.de/product/alfa-lora-halow-antenna-868mhz-5-dbi) |

> **Note on the "HaLow" label:** This is purely marketing copy from ALFA. The antenna has no relation to IEEE 802.11ah (Wi-Fi HaLow) — it is a standard 868 MHz LoRa stick antenna. The "HaLow" branding refers to the frequency band aesthetically, nothing more.

---

### Coax: U.FL → N-Female RG178 Pigtail

| Spec | Value |
|------|-------|
| Cable type | RG178 |
| Connectors | U.FL (RAK 4631 side) → N-Female (chassis bulkhead) |
| Length | < 20 cm |
| Estimated loss @ 868 MHz | ~0.3–0.5 dB per 30 cm |
| Impedance | 50 Ω |
| Source | [AliExpress](https://de.aliexpress.com/item/1005008877279360.html) |

Keeping the pigtail short matters with RG178 — it's a thin, flexible cable optimised for tight spaces, not long runs. Under 20 cm the insertion loss is negligible. The N-Female bulkhead provides a weatherproof, mechanically robust connection point for the external antenna without putting any stress on the board-level U.FL connector.

---

### Solar Panel: 10 W / 5 V

| Spec | Value |
|------|-------|
| Power | 10 W |
| Output voltage | 5 V |
| Original connector | Micro USB (replaced) |
| Modified connector | JST 1.25mm 2-pin |
| Connected to | RAK 19007 solar input (directly to charge controller) |
| Source | [AliExpress](https://de.aliexpress.com/item/1005008564961293.html) |

The panel originally shipped with a Micro USB plug, which was removed and replaced with a JST 1.25mm 2-pin connector to match the RAK 19007 solar input directly. This routes power straight to the charge controller — no USB negotiation layer, no additional connector to corrode outdoors.

10 W at 5 V is generously oversized for this build. The nRF52840 + SX1262 stack draws well under 150 mA peak during TX. Even on a grey, cloudy day the panel keeps the cell topped up with headroom to spare.

![Top-down view of the completed assembly: 10 W solar panel face-up with the ALFA 5 dBi antenna mounted at the top edge](/posts/2026/Building-a-Solar-Powered-MeshCore-Repeater-with-RAK-WisBlock/IMG_0025.JPEG)

---

## RF Chain Summary

For reference, the complete RF path from SoC to air:

```
RAK 4631 (SX1262) → U.FL → RG178 pigtail (<20 cm) → N-Female bulkhead → N-Male antenna cable → ALFA 5 dBi stick
```

Estimated total insertion loss between SoC output and antenna feed point: **< 1 dB**.

---

## Power Budget (rough)

| State | Current draw |
|-------|-------------|
| Deep Sleep | ~3–5 µA |
| RX (listening) | ~5–7 mA |
| TX @ +22 dBm | ~120 mA (short bursts) |
| Average (repeater duty) | ~10–20 mA |

At an average of ~15 mA and 3500 mAh capacity, the battery alone covers roughly **230 hours** of runtime — nearly 10 days without any sun. The 10 W panel makes this effectively indefinite under normal conditions.

![Sealed enclosure from the side: weatherproof IP-rated box with stainless steel clasps, solar panel mounted flat, and the ALFA antenna at the top — ready for outdoor deployment](/posts/2026/Building-a-Solar-Powered-MeshCore-Repeater-with-RAK-WisBlock/IMG_0026.JPEG)
