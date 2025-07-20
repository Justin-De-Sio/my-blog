---
title: "Homelab Adventures: The Beginning"
summary: "One server crammed in a TV stand, containers running wild, and GitOps keeping everything under control."
categories: ["Post", "Blog"]
date: 2025-04-23
draft: false
repo: "https://github.com/justin-de-sio/homelab"
series: ["Homelab Chronicles"]
---

It has been a while since I wanted to start a homelab to work on my personal projects.  
I had some old hardware lying around, so I figured, why not recycle it to build a little home server?

The idea was to run a Kubernetes cluster with some self-hosted apps, following a **DevSecOps** and **GitOps** mindset.  
I will use **Flux CD** and **Helm** to manage my deployments and automate everything properly from the start.

---

## Building the Homelab

- **Processor:** Intel i5-7600K 4.2GHz, 4 cores  
- **Memory:** 8 GB DDR4 RAM  
- **Storage:** 500 GB NVMe SSD  
- **Network:** 1 Gbps  
- **Case:** None — just raw power

![Photo of my server](homelab.jpeg)

Yes, that's a server crammed into a TV shelf.  
Yes, it gets a little hot.  
Is it up to code? Absolutely not.  
Is it safe? Probably not.  
Am I proud? Hell yeeeah. 🔥😎

The server is connected to a small switch linked to my router about 5 meters away.  
Thanks to that, I can properly hook up my homelab, TV, and Freebox Player without monopolizing all the ports on my router.  
Everything fits into the same piece of furniture — it's discreet, practical, and budget-friendly.

---

## Experience feedback  

I installed **Ubuntu Server** and **K3s** on the machine.  
At the beginning, I had set up **Talos Linux**, a production-grade Kubernetes-focused distribution. But I found it too strict and abstract for learning purposes.  
My current stack with **Ubuntu Server** and **K3s** allows me to move forward smoothly while keeping great flexibility.

I've already learned an important lesson: sometimes, **starting simple** is the best way to learn.

---

## Git Repo  

Everything is versioned here if you want to take a look or get inspired:  
[See the GitHub repo 🚀](https://github.com/Justin-De-Sio/homelab)
