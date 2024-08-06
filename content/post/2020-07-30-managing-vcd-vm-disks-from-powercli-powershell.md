---
title: Managing VCD VM Disks from PowerCLI / PowerShell
author: Jon Waite
type: post
date: 2020-07-30T04:28:32+00:00
url: /2020/07/managing-vcd-vm-disks-from-powercli-powershell/
categories:
  - PowerCLI
  - PowerShell
  - vCloud Director
  - VMware
tags:
  - PowerCLI
  - PowerShell
  - vCloud Director
  - VMware

---
One of the issues I come up against every now and again is when a customer asks how they can automate some functionality in our VMware Cloud Director (VCD) cloud platform and I think &#8216;that should be easy in PowerShell / PowerCLI&#8217; only to find out that the necessary cmdlets to accomplish the activity aren&#8217;t present.

Such was this case where a customer needed a way to manage the internal hard disks attached to some of their virtual machines from code. While I typically use the awesome [Terraform provider for VCD][1] these days which can easily cope with this scenario, in this case there was a requirement to manage VM hard disks from within existing PowerShell scripts.

So, as usual, I set about writing an extension module for PowerCLI to add this functionality. I also wanted to challenge myself to write some &#8216;better&#8217; PowerShell code that would do automatic detection of VCD API versions (and so work with future releases and API versions without having to be updated each time). I also wanted to handle XML from the VCD API &#8216;properly&#8217; &#8211; using the PowerShell XML methods, namespaces and properties and avoid simply &#8216;string bashing&#8217; XML fragments together.

The result of all of this is a module I&#8217;ve published today on Github and in [PowerShell Gallery][2] &#8216;CIVMDisks&#8217;. I&#8217;ve written up reasonably comprehensive details and examples in the Github repository so rather than repeat all of that here I&#8217;ll just include a [link][3].

The module contains 4 cmdlets:

<table class="aligncenter" style="height: 99px; width: 100%; border-collapse: collapse; border-style: none; border-color: #0000ff; background-color: #ffffff;" border="1" cellspacing="1" cellpadding="1">
  <tr>
    <td style="width: 30.1531%;">
      Get-CIVMDisk
    </td>
    
    <td style="width: 49.3522%;">
      Get the details of existing internal disks on a VCD VM
    </td>
  </tr>
  
  <tr>
    <td style="width: 30.1531%;">
      Add-CIVMDisk
    </td>
    
    <td style="width: 49.3522%;">
      Add a new disk (and controller if required) to a VCD VM
    </td>
  </tr>
  
  <tr>
    <td style="width: 30.1531%;">
      Update-CIVMDiskSize
    </td>
    
    <td style="width: 49.3522%;">
      Resize an existing VCD VM disk (only increases permitted)
    </td>
  </tr>
  
  <tr>
    <td style="width: 30.1531%;">
      Remove-CIVMDisk
    </td>
    
    <td style="width: 49.3522%;">
      Remove/delete a disk from a VCD VM
    </td>
  </tr>
</table>

**As always, please test properly in a test/dev environment before using this in production**, and feel free to log issues and/or PRs against the github repo if you find problems.

I also want to thank Eugene Burhovetsky from the VMware Cloud Providers Slack channel for doing some early testing and helping catch a bug in handling invalid SSL certificates for me.

Hopefully this will be useful for some of you.

Jon.

 [1]: https://www.terraform.io/docs/providers/vcd/index.html
 [2]: https://www.powershellgallery.com/packages/CIVMDisks
 [3]: https://github.com/jondwaite/civmdisks