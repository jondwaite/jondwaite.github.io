---
title: Installing Microsoft Azure Stack TP2 on VMware ESXi
author: Jon Waite
type: post
date: 2016-09-30T21:51:08+00:00
url: /2016/10/installing-microsoft-azure-stack-tp2-on-vmware-esxi/
categories:
  - Azure Stack
  - Microsoft
  - VMware
tags:
  - Azure
  - Azure Stack
  - ESXi
  - Installation
  - Microsoft
  - Troubleshooting
  - VMware
  - vSphere 6

---
This week Microsoft released Technical Preview 2 (TP2) of their ‘Cloud in a box’ Azure Stack product. This is scheduled for release in mid-2017 to allow enterprises and service providers to run Azure consistent services from their own datacenters.

TP2 has a number of additional features over TP1 released earlier this year, but doesn’t support installation as a virtual machine. The hardware requirements are detailed [here][1] but basically you’re going to need a reasonably good spec server with enough local hard disks to be able to install it.

As I had good success running the previous TP1 release of Azure Stack in a virtual machine I thought I’d see if the same could be done with TP2. As with TP1, installation is only supported for a single machine node (clustered multi-node deployments are likely to come with TP3).

Of course installing TP2 as a VM is completely unsupported by both Microsoft and VMware so **please don’t bug them with any issues** – since TP2 is definitely not for production use this shouldn’t be a huge concern.

After several failed attempts I finally worked out a method to allow installation of TP2 as a virtual machine using VMware ESXi 6.0 Update 2 as the hypervisor platform. The key is in building the host virtual machine correctly and in modifying a couple of places in the installation PowerShell scripts to bypass the checks for physical hardware.

To start, create a new virtual hardware v11 Windows VM with appropriate sizing (I used a 200GB system disk, 128GB of RAM and 12 CPU cores configured as a single socket / 12 cores arrangement).

I then made the following changes:

  * Add a new SCSI host bus adapter and set the ‘bus sharing’ for this adapter to ‘Physical’ – this is required to allow the Storage Spaces Direct (S2D) configuration in the Azure Stack installer to correctly configure clustered storage.
  * Make sure that ‘Expose hardware assisted virtualization to the guest OS’ option is enabled to allow the VM to run the Hyper-V role and nested VMs.
  * Add 4 new virtual hard disks of at least 150GB size each (I used 200GB for each disk again) and configure these as ‘Thick provisioned eager zeroed’ and make sure they are attached to the new (physical bus sharing) SCSI adapter.
  * Use a single VMXNET3 network adapter connected to a network that has a DHCP server available on it.
  * Change portgroup security for the network to which the VM is attached to allow ‘Promiscuous Mode’, ‘MAC address changes’ and ‘Forged Transmits’.
  * Set the VM to boot to BIOS on next power up and when it boots make sure to set the BIOS date/time to match your current timezone date/time.

Next power on and install a base operating system on the VM (I used Server 2012 R2, but it really doesn’t matter as this environment is only used to bootstrap the installation process).

Once the server is running, download  and unpack Azure Pack TP2 and move the extracted ‘CloudBuilder.vhdx’ file to the root of the C:\ drive. Following the Microsoft instructions to download and run the ‘PrepareBootFromVHD.ps1’ script which will reconfigure the VM to boot from the CloudBuilder.vhdx file and restart the VM.

Note: Depending on disk speed It can take a considerable time to extract CloudBuilder.vhdx from the TP2 archive – you might want to keep a copy of it elsewhere on your network (or on the VM disk if you have space) in case you need to restart the installation from scratch.

Once the VM is up and running from the TP2 CloudBuilder vhdx image, make the following changes:

  * Install VMware Tools (required to add the VMXNET3 network driver) and restart when prompted. - See note below, E1000 network adapter may be a better choice.
  * (Optional) Rename the computer and restart when prompted.
  * (Optional) Change the VM’s IP address to a static IPv4 address (rather than just using DHCP) so you can easily locate it on the network later – note that DHCP is still a requirement for the other VMs unless you use the Microsoft documented installer switches to allow use of a static addresses.
  * Make sure that the date/time and timezone are set correctly and match the VM BIOS setting (Can’t stress this enough, I had at least 3 failed installation attempts due to date/time problems).
  * Make the following changes to the C:\CloudDeployment\Roles\PhysicalMachines\Tests\BareMetal.Tests.ps1 file:

Line 376:  
Change:  
`$physicalMachine.IsVirtualMachine | Should Be $false`

To:  
`$false | Should Be $false`

Line 453:  
Change:  
`($physicalMachine.Processors.NumberOfEnabledCores | Measure-Object -Sum).Sum | Should Not BeLessThan $minimumNumberOfCoresPerMachine`

To:  
`12 | Should Not BeLessThan $minimumNumberOfCoresPerMachine`

Then save the file. (The second change should not be necessary if you’ve built the VM with at least 12 cores, but the installation script appears to detect the number of physical cores as ‘0’ in a VM so this is required).

You should now be able to run the CloudDeployment\Configuration\InstallAzureStackPOC.ps1 script and everything should work…..

The installation process will take a considerable time, but hopefully if you’ve configured everything correctly you’ll have a working Azure Stack TP2 installation at the end with all of the required infrastructure servers running as Hyper-V guests within the VM.

NOTE: I hit an issue with installation failing at step 60.61.93 and thought this was related to installing in a VM, but it appears this is a more general issue with TP2 installation – see [this][2] MSDN thread for possible solutions if you encounter this error. If you encounter any issues with the installation I also recommend following the troubleshooting advice [here][3].

Best of luck trying this out for yourselves!

Update 7th Oct 2016: If you're having issues with guest (nested Hyper-V) VMs crashing, try using the E1000 network adapter for the host instead of VMXNET3, I've been doing some testing with this and E1000 may be a better option and prevent this occurring.

Jon

 [1]: https://azure.microsoft.com/en-us/documentation/articles/azure-stack-deploy/
 [2]: https://social.msdn.microsoft.com/Forums/en-US/e36ca571-b38d-4098-8ed1-39e3f906f6c2/azure-stack-tp2-deployment-error?forum=AzureStack
 [3]: https://azure.microsoft.com/en-us/documentation/articles/azure-stack-rerun-deploy/