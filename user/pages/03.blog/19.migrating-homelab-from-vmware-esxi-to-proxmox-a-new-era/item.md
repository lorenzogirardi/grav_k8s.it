---
title: 'Migrating Homelab from VMware ESXi to Proxmox: A New Era'
date: '2025-04-28 21:35'
taxonomy:
    tag:
            - cluster
            - containers
            - lxc
            - proxmox
            - vmware
---

### Table of Contents

  *   * Introduction
  * Why I Moved from ESXi to Proxmox
    * 1\. Cost and Licensing Changes
    * 2\. Hardware Compatibility
    * 3\. Customization and Flexibility
    * 4\. Open Ecosystem
  * The Migration Approach: How I Did It
    * Step 0: Hardware comparison
    * Step 1: Backup Everything
    * Step 2: Prepare the New Environment
    * Step 3: Convert and Import VMs
    * Step 4: Testing
  * Day-to-Day Differences: ESXi vs Proxmox
  * Things You Might Miss from VMware (At First)
  * Bonus: Things I Gained with Proxmox
  * Conclusion



## 

## Introduction

For years, VMware ESXi has been the backbone of my homelab. Solid, reliable, and well-documented, it was a natural choice.  
But the landscape has changed. Since Broadcom's acquisition of VMware, the free ESXi license has been discontinued, and support for non-enterprise hardware — especially MiniPCs and NUCs — has become a challenge.  
The writing was on the wall: it was time for something new.  
Enter **Proxmox** — an open-source alternative that’s powerful, flexible, and completely free.

In this post, I’ll walk you through why I migrated, how I did it, and how life with Proxmox compares to my old ESXi setup.

##   
Why I Moved from ESXi to Proxmox

### 1\. **Cost and Licensing Changes**

  * **ESXi Free License Gone** : Post-Broadcom, ESXi free license is no longer offered.

  * **Proxmox** : 100% open-source. Optional support subscriptions are available but not required.




### 2\. **Hardware Compatibility**

  * **ESXi** : Compatibility issues with newer or non-server-grade hardware (MiniPCs, NUCs, etc.).

  * **Proxmox** : Based on Debian Linux, excellent support for a wide range of consumer and prosumer hardware.




### 3\. **Customization and Flexibility**

  * ESXi once relied on tools like `[ESXi-Customizer](https://www.v-front.de/p/esxi-customizer.html)` to inject missing drivers — which are now broken or unsupported.

  * Proxmox, thanks to Linux's modular driver support, works _out of the box_ or is easily tunable.




### 4\. **Open Ecosystem**

  * Proxmox integrates smoothly with **ZFS** , **LXC containers** , **KVM/QEMU** virtual machines, **native clustering** , and **high availability** — without costly add-ons.




* * *

## The Migration Approach: How I Did It

Migrating from ESXi to Proxmox was surprisingly smooth — but it needs careful planning.

### Step 0: Hardware comparison

My solution was based on a intel nuc intel j5005 bought in 2019  
the same configuration you can see in this post   
<https://www.k8s.it/homelab-when-small-is-big.html>  
  
With the opportunity to refresh my hardware i started a check with the goal to have more power , more ram with almost the same power consumption

![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/Screenshot-2025-04-28-at-21.13.27.png)

so i bought a couple of new mini pc with intel n95 and ryzen 5500u based on availability on used market to stay under the 150$ each (intel was bought for 100)

both are supporting 32gb ram , ryzen can reach up to 64  
  


### Step 1: Backup Everything

  * **Backup VMs** in ESXi first (i used [ghettovcb](https://github.com/lamw/ghettoVCB))




### Step 2: Prepare the New Environment

  * **Install Proxmox** on new hardware 

  * Configure Proxmox basics: network, storage, users , monitoring 


![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/Screenshot-2025-04-28-at-21.19.42.png)![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/Screenshot-2025-04-28-at-21.20.22.png)![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/Screenshot-2025-04-28-at-21.22.39.png)![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/screencapture-services-k8s-it-grafana-d-kxQQuHRZks-proxmox-2025-04-28-21_23_50.png)

### Step 3: Convert and Import VMs

You can’t just "drop" an ESXi VM onto Proxmox. You have a few options:

  * Export your VMs as **OVF/OVA** and then import them (long long long process)

  * Or use automatic or manual tools provided by proxmox (**suggested**) <https://pve.proxmox.com/wiki/Migrate_to_Proxmox_VE#Migration>

  * Recreate the VM configuration manually (CPU, memory, NICs) in Proxmox and attach the converted disk.




### Step 4: Testing

  * Most of the probles i had , were related to centos vms with vmware tools that "breaks" the kernel in proxmox (just regenerated initramfs or install the latest kernel withous vmware tools)




##   
Day-to-Day Differences: ESXi vs Proxmox

Feature| VMware ESXi| Proxmox VE  
---|---|---  
**UI**|  Polished but proprietary| Functional, open, web-based  
**Storage**|  Datastores (VMFS)| ZFS, Ceph, LVM, Directory-based  
**Backup**|  Add-on required (e.g., Veeam)| [Built-in Backup/Restore system](https://gist.github.com/lorenzogirardi/2337bdc557f9ec5e0cce7877b022fd45)  
**Snapshots**|  Basic, no schedules (free version)| Scheduled, ZFS-native snapshots  
**Cluster Setup**|  Requires vCenter (paid)| Built-in, easy 1-liner commands  
**Container Support**|  No native container support| Native LXC container management  
**Networking**|  Standard virtual switches| Linux bridges, OVS if needed  
**Monitoring**|  Basic| Built-in metrics + optional Grafana integration  
**Community**|  Strong but moving corporate-focused| Growing and very active  
  
##   
Things You Might Miss from VMware (At First)

  * **vSphere Web UI polish** : Proxmox UI is solid but a bit utilitarian compared to VMware’s.

  * **Driver/Hardware Stability for Enterprise Gear** : VMware still wins if you're running enterprise-grade servers.

  * **Super easy live migrations** (though Proxmox clustering is powerful once set up).




##   
Bonus: Things I Gained with Proxmox

  * **Move some workload from virtual machine to lxc****** to reduce resources and increase performances 

  * **Native backup and replication** at no additional cost.

  * **Access to underlying Linux** : Install packages, scripts, fine-tune, or debug without limitations.

  * **More frequent updates** without heavy licensing discussions.




##   
Current installation

![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/Screenshot-2025-04-28-at-21.33.13.png)

[![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/Screenshot-2025-04-28-at-21.41.51-thumbnail.webp)](<https://www.k8s.it/media/posts/46/gallery/Screenshot-2025-04-28-at-21.41.51.png>)[![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/Screenshot-2025-04-28-at-21.41.40-thumbnail.webp)](<https://www.k8s.it/media/posts/46/gallery/Screenshot-2025-04-28-at-21.41.40.png>)[![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/Screenshot-2025-04-28-at-21.41.18-thumbnail.webp)](<https://www.k8s.it/media/posts/46/gallery/Screenshot-2025-04-28-at-21.41.18.png>)[![](/user/images/migrating-homelab-from-vmware-esxi-to-proxmox-a-new-era/Screenshot-2025-04-28-at-21.41.07-thumbnail.webp)](<https://www.k8s.it/media/posts/46/gallery/Screenshot-2025-04-28-at-21.41.07.png>)

The decision to move from VMware ESXi to Proxmox wasn't easy — but it was the right one for me and my homelab.

Not only have I regained flexibility and future-proofed my setup, but I’ve also opened the door to new technologies like ZFS, LXC, and native clustering — all for free and without vendor lock-in.

If you’re on the fence, **test it yourself** : spin up a Proxmox node, migrate a test VM, and feel the difference.  
The open-source homelab future is here — and it’s called **Proxmox**.
