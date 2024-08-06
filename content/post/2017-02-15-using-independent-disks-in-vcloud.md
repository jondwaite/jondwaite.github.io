---
title: Using Independent Disks in vCloud
author: Jon Waite
type: post
date: 2017-02-14T21:28:05+00:00
url: /2017/02/using-independent-disks-in-vcloud/
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
[Yesterday][1] I wrote about the PowerShell module I&#8217;ve written ([CIDisk.psm1][2]) to allow manipulation of independent disks in a vCloud environment. This post shows some usage options and details some of the caveats to be aware of when using disks in this manner.

My test environment has two VMs (named imaginatively &#8216;vm01&#8217; and &#8216;vm02&#8217;), and the VDC they are in has access to four different storage profiles (&#8216;Platinum&#8217;, &#8216;Gold&#8217;, &#8216;Silver&#8217; and &#8216;Bronze&#8217; storage). The default storage policy for the VDC is &#8216;Bronze&#8217;, but what if we want to create independent disks on other profiles? The -StorageProfileHref parameter to New-CIDisk lets us do this. Once connected to our cloud (Connect-CIServer) we can find the Hrefs of the available storage profiles we can use:

<pre class="lang:ps decode:true ">C:\&gt; $vdc = Get-OrgVdc -Name '&lt;My VDC Name&gt;'

C:\&gt; $vdc.ExtensionData.VdcStorageProfiles.VdcStorageProfile | Select Name, Href

Name                     Href                                                                                       
----                     ----                                                                                       
Platinum Storage Profile https://my.cloud.com/api/vdcStorageProfile/4777f1a3-1f71-4e47-831e-fc7ed80376c3
Silver Storage Profile   https://my.cloud.com/api/vdcStorageProfile/d14c6c2e-2cfb-4ffe-9f31-599c3de42150
Bronze Storage Profile   https://my.cloud.com/api/vdcStorageProfile/dc382284-bc65-4e61-b85f-ea7facba63e4
Gold Storage Profile     https://my.cloud.com/api/vdcStorageProfile/f89df73a-2fa5-40e4-9332-fdd6e29d36ac</pre>

Let&#8217;s create 2 independent disks, a 10G disk on &#8216;Platinum&#8217; storage and a 100G disk on &#8216;Silver&#8217; storage:

<pre class="lang:ps decode:true ">C:\&gt; New-CIDisk -DiskName 'disk01-plat' -DiskSize 10G -StorageProfileHref https://my.cloud.com/api/vdcStorageProfile/4777f1a3-1f71-4e47-831e-fc7ed80376c3 -DiskDescription 'Platinum test disk'
Request submitted, waiting for task to complete...
Task completed successfully.

Name        : disk01-plat
Href        : https://my.cloud.com/api/disk/a1b00fa4-3d84-4c75-bc76-7604b96cdcab
Description : Platinum test disk
Size        : 10 GB
BusType     : lsilogicsas
Storage     : Platinum Storage Profile
AttachedTo  : Not Attached

C:\&gt; New-CIDisk -DiskName 'disk02-silv' -DiskSize 100G -StorageProfileHref https://my.cloud.com/api/vdcStorageProfile/d14c6c2e-2cfb-4ffe-9f31-599c3de42150 -DiskDescription 'Silver test disk'
Request submitted, waiting for task to complete...
Task completed successfully.

Name        : disk02-silv
Href        : https://my.cloud.com/api/disk/b02b50fd-1305-454d-923c-26ef9874a3fd
Description : Silver test disk
Size        : 100 GB
BusType     : lsilogicsas
Storage     : Silver Storage Profile
AttachedTo  : Not Attached</pre>

We can see in the vCloud interface that these disks now exist in our VDC (Note: you may have to completely refresh your vCloud session using your browser&#8217;s refresh before the &#8216;Independent Disks&#8217; tab appears):

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-142" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-001.png" alt="" width="1186" height="174" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-001.png 1186w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-001-300x44.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-001-768x113.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-001-1024x150.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-001-250x37.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-001-150x22.png 150w" sizes="(max-width: 1186px) 100vw, 1186px" /> 

There are no context actions for these disks though and we can&#8217;t attach/detach them to VMs in the vCloud interface.

Our VM01 virtual machine currently has a 40GB base disk attached and no other storage:

&nbsp;

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-143" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-002.png" alt="" width="759" height="192" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-002.png 759w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-002-300x76.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-002-250x63.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-002-150x38.png 150w" sizes="(max-width: 759px) 100vw, 759px" /> 

We can mount both our new independent disks to this VM using the following:

<pre class="lang:ps decode:true ">C:\&gt; $vm01 = Get-CIVM -Name 'vm01'

C:\&gt; $disk01 = Get-CIDisk -DiskName 'disk01-plat'

C:\&gt; $disk02 = Get-CIDisk -DiskName 'disk02-silv'

C:\&gt; Mount-CIDisk -VMHref $vm01.Href -DiskHref $disk01.Href
Request submitted, waiting for task to complete...
Task completed successfully.

C:\&gt; Mount-CIDisk -VMHref $vm01.Href -DiskHref $disk02.Href
Request submitted, waiting for task to complete...
Task completed successfully.</pre>

Looking at the VM01 Hardware tab following this shows both disks mounted:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-144" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-003.png" alt="" width="984" height="712" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-003.png 984w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-003-300x217.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-003-768x556.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-003-207x150.png 207w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-003-150x109.png 150w" sizes="(max-width: 984px) 100vw, 984px" /> 

Note again that no manipulation options are available in the vCloud UI, but at least it&#8217;s obvious that independent disks have been attached to VM01.

After rescanning storage in the guest, we can see the new storage devices on VM01:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-145" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-004.png" alt="" width="758" height="233" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-004.png 758w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-004-300x92.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-004-250x77.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-004-150x46.png 150w" sizes="(max-width: 758px) 100vw, 758px" /> 

And once these are brought online, initialized, storage volumes created and drive letters assigned, we can use the disks inside the guest (the volume names don&#8217;t get automatically mapped &#8211; I&#8217;ve just named the volumes the same as the independent disk objects for consistency):

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-146" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-005.png" alt="" width="535" height="152" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-005.png 535w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-005-300x85.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-005-250x71.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-005-150x43.png 150w" sizes="(max-width: 535px) 100vw, 535px" /> 

At this point everything appears to be working fine, but there can be a catch here &#8211; if you restart the virtual machine you may find that the server attempts to boot from one of the newly mounted independent disks. Luckily vCloud Director 8.10 allows us to get into the VM BIOS and change the boot order settings:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-147" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-006.png" alt="" width="983" height="712" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-006.png 983w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-006-300x217.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-006-768x556.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-006-207x150.png 207w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-006-150x109.png 150w" sizes="(max-width: 983px) 100vw, 983px" /> 

Once restarted into BIOS we can select the correct boot order:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-148" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-007.png" alt="" width="638" height="481" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-007.png 638w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-007-300x226.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-007-199x150.png 199w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-007-150x113.png 150w" sizes="(max-width: 638px) 100vw, 638px" /> 

With the server restarted, we can create some test content in &#8216;disk01-plat&#8217; to prove that the data moves when we reattach this disk to VM02:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-149" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-008.png" alt="" width="679" height="417" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-008.png 679w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-008-300x184.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-008-244x150.png 244w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-008-150x92.png 150w" sizes="(max-width: 679px) 100vw, 679px" /> 

And to dismount &#8216;disk01-plat&#8217; from VM01 and mount it to VM02 we can:

<pre class="lang:ps decode:true ">C:\&gt; Dismount-CIDisk -VMHref $vm01.Href -DiskHref $disk01.Href
Request submitted, waiting for task to complete...
Task completed successfully.

C:\&gt; $vm02 = Get-CIVM -Name 'vm02'

C:\&gt; Mount-CIDisk -VMHref $vm02.Href -DiskHref $disk01.Href
Request submitted, waiting for task to complete...
Task completed successfully.</pre>

Looking at the available storage in VM02 after a disk rescan shows our disk has transfered across:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-150" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-009.png" alt="" width="1036" height="411" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-009.png 1036w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-009-300x119.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-009-768x305.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-009-1024x406.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-009-250x99.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-009-150x60.png 150w" sizes="(max-width: 1036px) 100vw, 1036px" /> 

Finally, checking the contents of the &#8216;E:\&#8217; drive shows our test folder & file have made it across:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-151" src="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-010.png" alt="" width="1037" height="494" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-010.png 1037w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-010-300x143.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-010-768x366.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-010-1024x488.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-010-250x119.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/02/cidisk-010-150x71.png 150w" sizes="(max-width: 1037px) 100vw, 1037px" /> 

And Get-CIDisk can be used to verify the disk attachments after moving disk01 to VM02:

<pre class="lang:ps decode:true ">C:\&gt; Get-CIDisk | ft -AutoSize

Name        Href                                                               Description        Size   BusType     Storage                  AttachedTo
----        ----                                                               -----------        ----   -------     -------                  ----------
disk01-plat https://my.cloud.com/api/disk/a1b00fa4-3d84-4c75-bc76-7604b96cdcab Platinum test disk 10 GB  lsilogicsas Platinum Storage Profile vm02      
disk02-silv https://my.cloud.com/api/disk/b02b50fd-1305-454d-923c-26ef9874a3fd Silver test disk   100 GB lsilogicsas Silver Storage Profile   vm01</pre>

Hopefully this gives a better idea of how CIDisk can be used to manage independent disks in a vCloud environment, it would be nice if VMware included the management functions in the UI, but for now at least you can use PowerShell to easily achieve the same results without having to write against the API directly.

As always, any comments / feedback greatly appreciated.

Jon

 [1]: http://152.67.105.113/2017/02/independent-disks-in-vcloud-via-powercli/
 [2]: https://github.com/jondwaite/cidisk