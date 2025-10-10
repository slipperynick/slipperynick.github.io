---
title: "Homelabbing Part #1 — Low-Power Proxmox on the MeLE N100 w/ RunZero"
date: 2025-10-08 21:00 +0200
categories: [Homelab, Proxmox]
tags: [homelab, proxmox, mele-n100, runzero, low-power, vmware, dream-router, synology]
image: /assets/img/homelab/mele-n100-thumb.jpg
description: "Rebuilding my homelab after a few years away — this time on a low-power MeLE N100 mini PC running Proxmox VE."
---

After a few years away, I’ve started rebuilding my homelab again — this time with a focus on **simplicity, efficiency, and silence**.  
My new foundation is the **MeLE N100 mini PC**, a fanless, ultra-low-power box that’s been surprisingly capable so far.

---

## 🕰️ Looking Back — The Previous Homelab

Before this minimal setup, I used to run a larger, more complex lab. It was centered around the following:

- a **UniFi Dream Router** for networking (previously a netgate for pfSense)
- a **NUC8** for virtualization
- a **Synology NAS**.
- a few **LG monitors** (I prefer simplicity these days :D)
- the compute (mac and windows)

That setup included:
- VLAN segmentation between work and home Wi-Fi (so corporate and personal traffic never touched)

![Old homelab gear 1](/assets/img/posts/2025-10-08-homelab-part1/old-homelab1.jpg)

![Old homelab gear 2](/assets/img/posts/2025-10-08-homelab-part1/old-homelab2.jpg)


Fast-forward about three or four years, moving continents, and a lot of simplification later — I’m starting again, keeping things *as minimal as possible* this time around.

---

## 🖥️ The Hardware — MeLE N100 Mini PC

![MeLE N100 mini PC on desk](/assets/img/posts/2025-10-08-homelab-part1/homelab1.jpeg)

- Intel N100 CPU (4 cores)  
- 32 GB RAM  
- 512 GB NVMe SSD  
- Completely fanless and whisper-quiet  

Installing **Proxmox VE 9** from a bootable USB was straightforward — the BIOS recognized the stick instantly with `F7` at boot.

---

## 🧩 Proxmox Running Smoothly

![Proxmox VE dashboard](/assets/img/posts/2025-10-08-homelab-part1/homelab2.jpeg)

Once installed, Proxmox detected all hardware out of the box.  
My first VM is a **Debian 12** instance that serves as my main Linux environment and home for network and security tools.

---

## 🔋 Power Efficiency

![Smart-plug power stats showing ≈ 5 W idle](/assets/img/posts/2025-10-08-homelab-part1/homelab3.jpeg)

The system idles around **5 watts**, roughly the same as a single LED bulb — about €13 per year if left on continuously.  
Exactly what I was aiming for: a quiet, always-on platform that doesn’t waste energy. I recently put in some solar panels which cost money to send electric back to the grid. Hopefully this offsets that injustice :)

---

## 🔍 RunZero Explorer Installed

![RunZero scan results](/assets/img/posts/2025-10-08-homelab-part1/homelab4.jpg)

Inside the Debian VM, I installed **runZero Explorer** to rediscover everything on my LAN.  
Within minutes it identified my Proxmox host, NAS, and other devices — a great baseline for inventory and security mapping. I also realized it had been awhile since I used RunZero and it also has vulnerability information which it already helped me realize I need to patch a few of those IoT devices.

---

## 🤔 Next Steps

- Kubernetes cluster testing 
- Continue documenting / diagramming the *Homelab* adventures 


