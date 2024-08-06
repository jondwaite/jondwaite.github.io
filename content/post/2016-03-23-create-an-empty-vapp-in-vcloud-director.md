---
title: Create an empty vApp in vCloud Director
author: Jon Waite
type: post
date: 2016-03-23T03:03:14+00:00
url: /2016/03/create-an-empty-vapp-in-vcloud-director/
categories:
  - PowerShell
  - REST API
  - VMware
tags:
  - PowerCLI
  - PowerShell
  - REST API
  - vCloud Director
  - vCloud Director 8
  - VMware

---
Sometimes you just need to create a new vApp with no contents at all &#8211; maybe for testing, or maybe you want to populate it with VMs built &#8216;from scratch&#8217; rather than cloned from templates. This is easy to do in the vCloud Director web UI &#8211; you just skip the addition of any VM templates or new VMs and can easily create empty vApps, but how about programatically?

The VMware documentation is remarkably slim in this regard &#8211; all the documented methods I could find for vApp creation require either cloning from existing vApp templates, from existing VMs or from uploaded OVF files.

So how do we create a brand-new empty vApp? Turns out it&#8217;s pretty simple &#8211; once you discover the &#8216;composeVApp&#8217; method on an Organization VDC supports creation of empty vApps.

If using the REST API we can simply create an XML body document of type &#8216;composeVAppParams&#8217; and submit it against the OrgVDC&#8217;s /action/composeVapp link.

An example XML document body could be:

<span style="color: #0000ff;"><?xml version=&#8221;1.0&#8243; encoding=&#8221;UTF-8&#8243;?><br /> <ComposeVAppParams<br /> name=&#8221;MyEmptyVapp&#8221;<br /> xmlns=&#8221;http://www.vmware.com/vcloud/v1.5&#8243;<br /> xmlns:ovf=&#8221;http://schemas.dmtf.org/ovf/envelope/1&#8243;><br /> <Description>My vApp Description</Description><br /> <AllEULAsAccepted>true</AllEULAsAccepted><br /> </ComposeVAppParams></span>

We then &#8216;POST&#8217; this document body to the link: &#8216;https://<Cloud Server DNS name or IP address>>/api/vdc/<ID of our VDC>/action/composeVApp&#8217; not forgetting to add a header of &#8216;Content-Type: application/vnd.vmware.vcloud.composeVAppParams+xml&#8217; to the POST request.

If we want to accomplish the same thing using PowerShell / PowerCLI it&#8217;s easy too (once connected to our cloud using Connect-CIServer):

<span style="color: #0000ff;">$vapp = New-Object VMware.VimAutomation.Cloud.Views.ComposeVAppParams<br /> $vapp.Name = &#8220;MyEmptyVapp&#8221;<br /> $vapp.Description = &#8220;My vApp Description&#8221;<br /> $myorgvdc = Get-OrgVdc -Name &#8216;My OrgVDC Name&#8217;<br /> $myorgvdc.ExtensionData.ComposeVApp($vapp)</span>

No idea if this is &#8216;officially&#8217; supported or not &#8211; so use at your own risk and be aware that the implementation could change in a future release and break this (although I&#8217;d be surprised as this is almost certainly the action that the vCD web UI is submitting &#8216;behind the scenes&#8217; when you manually create an empty vApp).

Jon.