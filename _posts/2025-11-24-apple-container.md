---
title: "Playing with Apple’s New Container Tool"
date: 2025-11-24 11:00:00 +0100
categories: [Homelab, Apple]
tags: [Apple, Container, macOS, DevTools]
---

I’ve been experimenting with Apple’s new **Container** tool. It's a lightweight, Apple-native alternative to Docker. It’s still early days, but the idea is simple: fast, sandboxed app and service environments that integrate directly with macOS, Xcode, and the new privacy model Apple is pushing.

For now, I’m just walking through the **official tutorial**, getting a feel for how containers are built, run, and how the dev mode behaves. It’s surprisingly clean — no daemon, no Docker Desktop, and very fast on Apple silicon.

Here’s a short screen capture of me running the first few commands:

![Apple Container Recording](/assets/img/posts/2025-11-25-apple-container/container.gif){: .normal }

Shout out to asciinema + agg which made this recording extremely easy.