---
publish: true
title: Welcome!
created: 2026-10-28T23:24:19.526Z
modified: 2026-11-23T23:30:56.753Z
---

Welcome to my blog! Here I'll be posting notes from building, breaking, and fixing my homelab along with other technical projects: video game console repair and restoration, tinkering, IoT, study, and more.

---

# Homelab Projects

- [[Homelab Infrastructure Rebuild Overview]]
- [[Building my Blog]]

---

# Tools

A list of tools I use or have used. # links are tag pages, which will provide a list of all things I have written relevant to that topic.

#### Virtualization & Containers

- [Proxmox VE](https://www.proxmox.com/en/) - Virtualization host for my homelab environment to deploy and manage VMs #proxmox
- [Docker](https://www.docker.com/) - Containerization platform, all my services are defined as compose stacks
- [Portainer](https://www.portainer.io/) - Centralized Docker management platform where I deploy and manage containers and compose stacks across multiple Docker hosts with Portainer Agent

#### Networking & Security

- [OPNsense](https://opnsense.org/) - My router and firewall where I manage allowed traffic across subnets and hosts that are segmented with VLANs. #opnsense
- [WireGuard](https://www.wireguard.com/) - VPN service which I use to access and manage my homelab environment remotely
- [Unbound DNS](https://nlnetlabs.nl/projects/unbound/about/) - Recursive DNS resolver to manage my internal DNS
- [Traefik](https://traefik.io/traefik) - Reverse proxy that puts my internal services behind subdomains, with HTTPS from a Let's Encrypt wildcard certificate
- [Let's Encrypt](https://letsencrypt.org/) - Wildcard TLS certificate for internal services, issued through a Cloudflare DNS challenge with automatic renewal by Traefik
- [Cloudflare](https://www.cloudflare.com/) - Primary platform for deploying external services #cloudflare
  - DNS - Authoritative DNS for my domain, blueinblueeyes.org, and handles the DNS challlenge for Let's Encrypt
  - DDNS - Keeps the Wireguard endpoint pointed at my home IP
  - Tunnel - Allows me to host services from my DMZ without opening inbound ports
  - Proxy - Provides DDoS protection and hides my home IP for public services
- [Gluetun](https://github.com/passteque/gluetun) - VPN routing for containers

#### Identity & Secrets

- [Vaultwarden](https://github.com/dani-garcia/vaultwarden) - Centralized self-hosted password management, Bitwarden-compatible
- [YubiKey](https://www.yubico.com/) - Physical security key for MFA

#### Storage & Sync

- [Synology DSM](https://www.synology.com/en-global/dsm) - Synology operating system used to manage NFS and SMB shares
- [Syncthing](https://syncthing.net/) - Peer-to-peer file sync between my devices #syncthing

#### Monitoring

- [Beszel](https://www.beszel.dev/) - Lightweight server monitor application

#### Operating Systems

- [Debian](https://www.debian.org/) -  Base OS for my VMs
- [Ubuntu](https://ubuntu.com/) - OS for my retired singular Docker host
- [Linux Mint](https://linuxmint.com/) - My choice for Desktop Linux
- [Kali Linux](https://www.kali.org/) - Pen testing distro I use for testing my own network and learning

#### Imaging & Recovery

- [Ventoy](https://www.ventoy.net/en/index.html)  - Bootable USB tool that allows for multiple images on one drive #ventoy
- [Medicat USB](https://medicatusb.com/) - Pre-built USB toolkit for recovery and backup
- [Clonezilla](https://clonezilla.org/) - System imaging and recovery tool #clonezilla

#### Documentation & Diagrams

- [Obsidian](https://obsidian.md/) - Markdown based note-taking app where I write and keep my homelab documentation #obsidian
- [D2](https://d2lang.com/) - My preferred diagram as code
- [Draw.io](https://www.drawio.com/) - My preferred diagram solution that can be self-hosted

#### Publishing

- [Quartz](https://quartz.jzhao.xyz/) - Static site generator for converting markdown files to websites, designed with Obsidian in mind #quartz
- [Cloudflare Pages](https://www.cloudflare.com/products/pages/) - Cloudflare web hosting service #cloudflare

#### Media

- [Plex](https://www.plex.tv/watch-free/) - Self-hosted streaming platform for movies and TV shows
- [Calibre](https://calibre-ebook.com/) - eBook library management
- [Calibre-web](https://github.com/janeczku/calibre-web) - Hosted solution for the use of Calibre over the web, I primarily use it for the OPDS feature

#### Resources

- [selfh.st](https://selfh.st/) - Excellent resource for news, projects, and applications in the homelabbing space, with a curated self-hosted application directory, and an icon library for common services
- [linuxserver.io](https://www.linuxserver.io/) - Well-maintained Docker image library with consistent configurations and documentation
