---
title: 'Quickpost: Converting TrueNAS Scale VMs to VMware ESXi'
author: Jon Waite
type: post
date: 2023-02-04T01:14:17+00:00
excerpt: Quick post detailing how to convert TrueNAS Scale VMs held in zVol disks to VMDK format disk usable in VMware vSphere
url: /2023/02/quickpost-converting-truenas-scale-vms-to-vmware-esxi/
categories:
  - TrueNAS
  - VMware
tags:
  - Migration
  - qemu-img
  - TrueNAS Scale
  - VMware
  - vSphere

---
In my home lab I&#8217;ve used many different platforms over the years to host container and VM workloads, often I find I need to move VMs between different hypervisors. In this post I&#8217;ll detail the steps I used to move some virtual machines from TrueNAS Scale (which uses zVol storage for VM disks and KVM as a hypervisor to VMware ESXi).

<a rel="noreferrer noopener" href="https://www.truenas.com/truenas-scale/" target="_blank">TrueNAS Scale</a> is an awesome homelab storage option, and one that I&#8217;ve come increasingly familiar with over the last year or so. The zVol devices it uses to store VM disks are actually genuine block devices, not simply files in a filesystem as is common with other VM storage (e.g. VMFS and NFS). So to move disks between platforms we need to convert from the zVol to a VMDK format disk that ESXi understands.

In my homelab I&#8217;m using TrueNAS as a NFS datastore on my ESXi hosts which simplifies this type of conversion as I can simply specify the NFS path as the destination for the converted disk. If you don&#8217;t have NFS-mounted storage from TrueNAS then you can run the conversion to any accessible disk share and then &#8216;upload&#8217; the disk file to your VMware environment using the datastore browser.

You will need shell access to TrueNAS to run the following command, and write access to the path where the converted disk is to be created, in my homelab my VM volume is at the path /dev/zvol/ucs-ssd-int/VMs/dns01 and the NFS share is at /mnt/sata/nfs-cap-01, the command I run is therefore:

<pre class="wp-block-code"><code>qemu-img convert -f raw /dev/zvol/ucs-ssd-int/VMs/dns01 \
  -O vmdk -o adapter_type=lsilogic,subformat=streamOptimized,compat6 \
  /mnt/sata/nfs-cap-01/dns01.vmdk -p</code></pre>

The parameters here are reasonably self-explanatory:<figure class="wp-block-table">

<table>
  <tr>
    <td>
      -f raw
    </td>
    
    <td>
      Tells qemu-img that the source format is a &#8216;raw&#8217; disk image (which will be retrieved from the TrueNAS zVol path given &#8211; in this example /dev/zvol/ucs-ssd-int/VMs/dns01).
    </td>
  </tr>
  
  <tr>
    <td>
      -O vmdk
    </td>
    
    <td>
      Tells qemu-img that we want to write out in VMDK disk format.
    </td>
  </tr>
  
  <tr>
    <td>
      -o adapter_type&#8230;
    </td>
    
    <td>
      Specifies the destination disk adapter type, the streamOptimized creates &#8216;thin&#8217; format VMDK disk files.
    </td>
  </tr>
  
  <tr>
    <td>
      -p
    </td>
    
    <td>
      Shows progress as the disk is being converted (optional) &#8211; I find this useful for large disks that can take significant time to convert
    </td>
  </tr>
</table></figure> 

If your VM has multiple disks attached then the same command can be used to convert each of them (obviously change the output names for each disk so they don&#8217;t overwrite each other).

Once the disk has been converted, the next step is to create a new &#8216;placeholder&#8217; VM in vCenter (or on the ESXi host directly). One of the &#8216;gotchas&#8217; here is to make sure that the VM boot mode is configured the same as the source VM (BIOS or UEFI) otherwise the new VM won&#8217;t boot. TrueNAS Scale tends to create UEFI boot mode VMs by default and VMware typically creates BIOS boot mode VMs so this can catch you out if not careful.

I tend to create the VM in vCenter, remove any disks set to be auto-created, and set the appropriate boot mode to match the source VM.

I can then (via the Datastore browser in vCenter) move the converted disk image into the folder created for the new VM and add the converted vmdk file as an existing disk to the VM.

If everything has worked, the VM can now be powered on and will boot up fine. Check the networking for the VM as often in these types of migrations (especially with Linux VMs) the device name of the network adapter can change which can mean IP addresses may not be applied correctly and will need to be updated for the &#8216;new&#8217; network adapter. The same can also happen with bootloaders (e.g. grub on linux) with regards to disk device names so some additional work may be required to fix this up.

Next you can install VMware Tools (or open-vm-tools) in the guest OS after a successful migration (and remove any previous hypervisor assistance guest tools if installed).

Finally, if you have &#8216;autostart&#8217; configured in TrueNAS for the &#8216;source&#8217; VM then remember to turn that off or you could end up with two copies of the VM with the same IP address attempting to run at the same time which can lead to some very confusing issues.

As always, comments & feedback weldome,

Jon.