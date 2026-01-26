# My Homelab Adventures

## Purpose
This repository contains all things related to creating and operating a home lab. The goals of this project will be to
* Have a reliable playground for trying out new things
* Learn more about resource provisioning, security, networking, etc.
* Serve meaningful applications that allow me to own all the data
* Stop using work as a part-time hobby for some/all of the above :)

The first iteration of this was running [pihole](https://pi-hole.net/) on a [Raspberry Pi 3 B+](https://www.amazon.com/dp/B07BC7BMHY?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_1) that I had left over. That was super easy to set up and has been rock solid for 4+ years with essential no downtime. While it was fun to host that in a minimal system, it's time to upgrade into bigger and better things!

## Current System
### Hardware Selection
Since this is a playground, we're starting out _inexpensive_. The Pi is really fun to play around with, but it's way too light on resources to start running meaningful workloads. I am also a big fan of Kubernetes, so a miniature high availability cluster is the starting point. First task was to get new hardware. The selection criteria was pretty simple: look at Reddit (e.g. [r/homelab](https://www.reddit.com/r/homelab/)) to see what folks like to use. I wanted reliable hardware from major vendors in a platform that would be possible to upgrade (whenever RAM prices come down). A node was a good purchase candidate if was under $100, had a modern-ish multi-core PC, 8+ GB of DDR4 RAM, an SSD (preferably NVMe but SATA will do), power efficient, and quiet. Even single node clusters hate running on HDDs, so believe all those installation guides when they tell you to put (at least) etcd on an SSD.

Three machines have been acquired to server this purpose!

**Dell Optiplex 3050**
* 💰 Cost - $75
* 🏭 Processor - i5-6500T
* 💾 Storage - 480GB SATA SSD
* 🧠 Memory - 8GB DDR4

**Lenovo ThinkCentre M710Q**
* 💰 Cost - $75
* 🏭 Processor - i5-7500T
* 💾 Storage - 256GB NVMe SSD
* 🧠 Memory - 8GB DDR4

**HP EliteDesk 800 G3 Mini**
* 💰 Cost - $95
* 🏭 Processor - i5-7500T
* 💾 Storage - 256GB NVMe SSD
* 🧠 Memory - 8GB DDR4

All of these machines have decent options for expanding RAM and storage in the future. Importantly, I wanted three different machines to keep things flexible going forward in case any one of them had difficulty modifying the hardware. Some extra details about the server setup are available in `docs/server_setup.md`.

### Network Management
If we were just ripping a single node cluster, networking wouldn't be that big of a deal. We'd let our DHCP server running on the router give us an IP address on the home network and call it good. For multi-node, we need a little more stability. Following [KISS](https://en.wikipedia.org/wiki/KISS_principle) and because I can get decision paralysis, I didn't want to go all in on custom VLANs, firewalls, and the sort. The simplest solution here for getting stable IPs is to configure static IP addresses. This could be achieved by assigning static leases in the DHCP server or assigning static IPs directly with the servers' netplan configuration. Either would work fine, but I opted for static IPs. Last thing that had to be sorted out was to update the address pool of the DHCP server. The goal here is to make sure the dynamic pool never overlapped with my Kubernetes nodes and services. Since my default home LAN 192.168.1.0/24, I configured the DHCP to server only out of 192.168.1.2 through .199. This leaves me .200 to .255 for all my Kubernetes experimentation needs.

The netplan configuration used for a node is available in `src/netplan_templates`.

### Kubernetes
#### Provider
Since I wanted to get a little more familiar with Linux fundamentals, we're deferring on Kubernetes-specific operating systems like [Talos](https://www.talos.dev/). While [kubeadm](https://v1-34.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) is one of the more common ways to run a cluster, I want something lighter and easier to manage on my smaller hardware. There's a couple options here.
* [k3s](https://k3s.io/)
* [microk8s](https://canonical.com/microk8s)
* [kind](https://kind.sigs.k8s.io/)
* [k0s](https://k0sproject.io/)

I have a soft spot for standalone binaries, so that rules out microk8s. `snap` is weird anyway and I've generally only had issues with it (maybe that's just a vscode thing), so that's preferable anyway. `kind` is fun, but I don't want to deal with the overhead of Docker virtualization especially since I don't expect this to be ephemeral. I'm most familiar with `k3s` and have done some really fun things with it, so we're going to go with `k0s` as an opportunity to learn new things. If it doesn't work out, I'll write a doc explaining way and go back to comfy land of `k3s`.
