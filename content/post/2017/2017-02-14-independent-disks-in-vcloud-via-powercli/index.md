---
title: Independent Disks in vCloud via PowerCLI
author: Jon Waite
type: post
date: 2017-02-14T03:47:15+00:00
url: /2017/02/independent-disks-in-vcloud-via-powercli/
categories:
  - PowerShell
  - REST API
  - VMware
tags:
  - PowerCLI
  - PowerShell
  - REST API
  - vCloud Director

---
Another day, another customer requirement which I figured 'this will be easy' and turned out not to be quite so easy...

The customer in question is a tenant on our cloud platform and has built a VM to be their offline root Certificate Authority (CA). In line with their security practice, this VM has no network connectivity and is usually powered-off in their environment unless specifically required to issue or renew certificates.

They asked if there was an easy way to transfer certificate files issued by this VM to other servers in their infrastructure. In their (old) vSphere environment they would simply attach a new temporary virtual disk to the VM, copy the certificate files over and then attach the disk to the destination VM. Surely there had to be some similar functionality in vCloud Director?

Well, there's a bit of good and bad news on that...

By default disks in vCloud Director are assigned (permanently) to a VM, they can't be moved to different VMs. (That's the bad news). The good news is that vCD supports 'independent disks' which can be moved between VMs. The bad news is that this is an API-only operation (nothing in the web UI allows creation or manipulation of Independent disks, although you can see them if they exist). The worst news is that VMware PowerCLI even in the latest 6.5R1 version doesn't have any cmdlets to manipulate independent disks attached to vCloud VMs either.

So while I could have hacked something together to run directly against the vCloud Director REST API for this customer, I figured it would be better to have some reusable PowerShell cmdlets for this. So I set about writing some and I'm pleased to announce the first release of 'CIDisk', a collection of PowerShell cmdlets to manipulate independent disks in vCloud Director environments.

The module code, documentation and examples are now available on my github at <https://github.com/jondwaite/cidisk>

I'll do a followup post detailing some more advanced options and scenarios in the next day or two.

Edit - Followup post is now available [here][1].

As always I appreciate any/all feedback and hope someone else finds these useful.

Jon

 [1]: /2017/02/using-independent-disks-in-vcloud/