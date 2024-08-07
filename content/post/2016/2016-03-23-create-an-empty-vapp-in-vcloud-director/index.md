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
Sometimes you just need to create a new vApp with no contents at all - maybe for testing, or maybe you want to populate it with VMs built 'from scratch' rather than cloned from templates. This is easy to do in the vCloud Director web UI - you just skip the addition of any VM templates or new VMs and can easily create empty vApps, but how about programatically?

The VMware documentation is remarkably slim in this regard - all the documented methods I could find for vApp creation require either cloning from existing vApp templates, from existing VMs or from uploaded OVF files.

So how do we create a brand-new empty vApp? Turns out it's pretty simple - once you discover the 'composeVApp' method on an Organization VDC supports creation of empty vApps.

If using the REST API we can simply create an XML body document of type `composeVAppParams` and submit it against the OrgVDC's `/action/composeVapp` link.

An example XML document body could be:

```xml
<ComposeVAppParams name="MyEmptyVapp" xmlns="http://www.vmware.com/vcloud/v1.5" xmlns:ovf="http://schemas.dmtf.org/ovf/envelope/1">
<Description>My vApp Description</Description>
<AllEULAsAccepted>true</AllEULAsAccepted>
</ComposeVAppParams>
```

We then 'POST' this document body to the link: `'https://<Cloud Server DNS name or IP address>>/api/vdc/<ID of our VDC>/action/composeVApp'` not forgetting to add a header of `Content-Type: application/vnd.vmware.vcloud.composeVAppParams+xml` to the POST request.

If we want to accomplish the same thing using PowerShell / PowerCLI it's easy too (once connected to our cloud using Connect-CIServer):

```powershell
$vapp = New-Object VMware.VimAutomation.Cloud.Views.ComposeVAppParams
$vapp.Name = "MyEmptyVapp"
$vapp.Description = "My vApp Description"
$myorgvdc = Get-OrgVdc -Name 'My OrgVDC Name'
$myorgvdc.ExtensionData.ComposeVApp($vapp)
```

No idea if this is 'officially' supported or not - so use at your own risk and be aware that the implementation could change in a future release and break this (although I'd be surprised as this is almost certainly the action that the vCD web UI is submitting 'behind the scenes' when you manually create an empty vApp).

Jon.