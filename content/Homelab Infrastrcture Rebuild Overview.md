---
publish: true
created: 2026-11-02T23:28:24.848Z
modified: 2026-11-11T22:09:04.719Z
tags:
  - homelab
  - rebuild
---

In light of the wave of new security vulnerabilities in the previous weeks, I began to think a lot about how those could affect my homelab. In doing so, I tried to better document my homelab infrastructure so I could understand where I was at risk. This made me realize two things: I was potentially vulnerable in ways I hadn't considered, but also that my homelab is a mess, with a lack of consistency. This was due to the fact that I was applying what I was learning immediately, without a concept of best practice. Ultimately, I decided to recreate my entire homelab environment and its infrastructure, which I have been working on for the past few weeks. My goal was to recreate what I had but in a consistent, documented, and more secure way. Largely, I have accomplished this goal, but there always is still work to be done.

In the coming weeks I'll be sharing my knowledge (old and new) over topics like migration, virtualization, network segmentation with VLANs and subnets, VPNs, DNS, infrastructure-as-code, and personal digital security using tools such as OPNsense with Unbound, Proxmox, Wireguard, Docker, Traefik, and more.

I will be uploading my first post in this series about how I backed up my homelab, and installed and configured proxmox.

# Parts

1. [[Migrating to Proxmox]]
2. [[OPNsense Firewall Deployment, Configuration, and Segmentation with Proxmox]]
