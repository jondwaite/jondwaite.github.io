---
title: Client Integration Plugin madness
author: Jon Waite
type: post
date: 2016-02-23T23:56:49+00:00
url: /2016/02/client-integration-plugin-madness/
categories:
  - Client Integration
  - VMware
tags:
  - Client Integration
  - Plugins
  - vCloud Director 8
  - VMware
  - vSphere 6

---
One of the frustrations dealing with the vSphere Web Client has always been the requirement for a browser plugin to import/export OVF templates. In vSphere 6 and vCloud Director 8 this has reached a whole new level of frustration. The issue is that both vSphere 6 and vCloud Director 8 offer to download and install a package called 'VMware-ClientIntegrationPlugin-6.0.0.exe'. In an ideal world this plugin once correctly installed would work for the OVF import/export/upload/download functionality in both products&#8230; right?

Meanwhile in this world, although the package names are identical, the functionality is not - if you've installed the vSphere version then you can't upload/download OVFs or ISOs in vCloud Director and if you've got the vCD variant installed then vSphere OVF import/export doesn't work. Uninstalling and reinstalling the 'correct' version fixes the problem (until you need the 'other' one again), but can be easier said than done - particularly in the case of shared desktop server administration environments where other users having a browser session open will prevent reinstalling browser plugins.

So how do you tell the two packages apart? In the current releases of vSphere 6.0Update1 and vCloud Director v8.0 the packages are significantly different sizes:

|    |vCloud Director 8|vSphere 6.0 Update 1|
|---|---|---|
|Package Filename|VMware-ClientIntegrationPlugin-6.0.0.exe|VMware-ClientIntegrationPlugin-6.0.0.exe|
|File Version|11.0.0.2826|10.0.0.3637|
|Product Version|6.0.0.2826|6.0.0.3637|
|File Size|48.8 MB|94.9 MB|

So the easiest way to tell is the smaller 48.8 MB file is vCD and the larger 94.9 MB one is vSphere.

If (as I do) you often need to use both versions then maybe consider setting up separate management desktops (or virtual apps) for each so you can easily reach one that's going to work for you.

Hopefully VMware will fix this in a future release and provide a single integration plugin that works across both products.

**Update - 18th March 2016**

VMware have just released vCloud Director for Service Providers v8.0.1 (<a href="http://pubs.vmware.com/Release_Notes/en/vcd/801/rel_notes_vcloud_director_801.html" target="_blank">http://pubs.vmware.com/Release_Notes/en/vcd/801/rel_notes_vcloud_director_801.html</a>) which appears to have reverted the vCD Client Integration product to version 5.6.0 - there is also mention in the release notes on the possible clashes between vSphere and vCD client integration toolsets so it appears that VMware are at least aware of the issue.