---
title: 'Meshcore & Meshtastic'
ShowBreadCrumbs: false
comments: false
searchHidden: true
showToc: true
TocOpen: true
#tags: ["Meshcore", "Meshtastic", "LoRa"]
---

My activities around Meshcore LoRa mesh networking and Meshtastic projects.

## Overview

This page documents my projects, tests, and experiences with Meshcore and Meshtastic devices.

## Current Projects

### Home-Setup
**Description:**  
My current *Meshcore* setup at Koblenz. 

**Status:** Running

**Technical Details:**
| Parameter | Value |
|-----------|-------|
| Frequency | 868 MHz |
| Firmware  | v1.14.1 |


## Hardware Setup

Network Overview

```mermaid

graph TD
    subgraph "Koblenz Home Network"
        RB[Repeater<br/>DE-KO j3nstastic<br/>RAK 4631<br/>868 MHz<br/>20 dBm<br/>5 dBi Ant]
        
        CH[Companion Home<br/>j3nstastic home<br/>RAK 4631<br/>868 MHz<br/>3 dBi Ant]
        
        CM[Companion Mobile<br/>j3nstastic mobile<br/>SenseCAP T1000-E<br/>868 MHz<br/>3 dBi Ant]
    end
    
    RB -.->|LoRa Mesh<br/>Primary Relay<br/>~1.2km urban| CH
    RB -.->|LoRa Mesh<br/>Mobile Link<br/>~8km rural| CM
    CH -.->|LoRa Mesh<br/>Direct<br/>Backup Path| CM
    
    classDef repeater fill:#e1f5fe,stroke:#01579b,stroke-width:3px,color:#000
    classDef companion fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#000
    classDef mobile fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px,color:#000
    
    class RB repeater
    class CH companion
    class CM mobile
```

- Repeater: 
    - Name: [DE-KO j3nstastic](https://www.bytehero.io/posts/2026/meshcore-repeater-rak4631/)
    - Model: RAK 4631
- Companion #1 
    - Name: j3nstastic home
    - Model: RAK 4631
- Companion #2 
    - Name: j3nstastic mobile
    - Model: SenseCAP T1000-E

## Related posts

All [Meshcore & Meshtastic posts](/tags/lora/) (auto-generated list)



## Resources

- [Meshcore Documentation](https://github.com/meshcore-dev/MeshCore/blob/main/docs/faq.md)
- [Meshtastic Documentation](https://meshtastic.org/docs/)


## Contact

Questions about Meshcore/Meshtastic?  
Feel free to contact me at ***hello[at]bytehero.io***
