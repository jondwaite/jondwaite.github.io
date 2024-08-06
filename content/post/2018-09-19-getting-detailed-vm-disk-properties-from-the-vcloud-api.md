---
title: Getting detailed VM Disk Properties from the vCloud API
author: Jon Waite
type: post
date: 2018-09-19T01:05:38+00:00
url: /2018/09/getting-detailed-vm-disk-properties-from-the-vcloud-api/
categories:
  - PowerShell
  - REST API
  - vCloud Director
  - VMware
tags:
  - API
  - PowerCLI
  - PowerShell
  - vCloud Director

---
Since vCloud Director 8.10 VMware have allowed VMs to be created which have multiple disks using different storage policies. This can be very useful – for example, a database VM might have it’s database on fast storage but another disk containing backups or logs on slower/cheaper disk.

When trying to find out what storage is in use for a VM though this can create issues, the PowerCLI Get-CIVM cmdlet (and the Get-CIView cmdlet used to get extra information) aren’t able to properly report storage for VMs that consume multiple storage policies. This in turn can create problems for Service Providers when they need to report on overall VM disk usage divided by storage policy used.

As an example I’ve created a VM named ‘test01’ in a customer vDC which has 3 disks attached, the 2nd of these is on ‘Capacity’ tier storage while disks 1 and 3 are on ‘Performance’ storage. When we look at the VM details we see the following:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/09/image_thumb.png" alt="image" width="952" height="283" border="0" />][1]

Digging into the ExtensionData shows

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/09/image_thumb-1.png" alt="image" width="765" height="364" border="0" />][2]

The StorageProfile element looks like it may contain what we need, but unfortunately this only shows the ‘home’ Storage for the VM and doesn’t indicate that at least one of the VMs disks is on a different storage profile:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/09/image_thumb-2.png" alt="image" width="786" height="145" border="0" />][3]

After a lot of mucking around trying to find an easy way to discover the information, I ‘gave up’ and wrote a PowerShell module which accesses the vCD API directly to get the VM storage information (including storage tiers in use by each disk). The module isn’t overly efficient since it queries the storage profile reference for every disk on every VM (and so will result in a lot of calls if run for a large number of VMs), but otherwise works fine.

The module takes VM objects or a VM name as input and returns details on each disk attached to the VM including which storage profile they use. Save the script (e.g. as ‘Get-CIVMStorageProfile.psm1’) and then use ‘Import-Module .\Get-CIVMStorageProfile.psm1’ to import the function.

<pre class="lang:ps decode:true">&lt;#
  .Synopsis
   Gets detailed storage information from a vCloud VM.

.Description
   This function returns detailed disk information for a vCloud VM. Specifically
   it shows the number of disks attached and the storage policy assigned to each
   disk which is useful when VMs consume storage from multiple policies.

.Parameter CIVM
   The VM object (from Get-CIVM) to report storage for.

.Example
   Get-CIVM -Name 'test01' | Get-CIVMStorageDetail
#&gt;

Function Get-CIVMStorageDetail
{
     [CmdletBinding()]
     Param(
         [Parameter(ValueFromPipeline)]
         $CIVM
     )
     begin {}
     process
     {

        # API version to use when communicating with vCloud Director - API 27.0 is vCloud Director 8.20:
         $vCDAPIVersion = "27.0"
                 
         # Check if we've been passed a VM name or an actual VM object and handle appropriately
         if ($CIVM.GetType() -eq [String]) {
             try {
                 $VMObj = Get-CIVM -Name $CIVM -ErrorAction Stop
             } catch {
                 Write-Host -ForegroundColor Red "Error: Could not find a VM with the name $CIVM."
                 Break
             }
         }
         else {
             $VMObj = $CIVM
         }

        # Find our vCloud SessionId that matches the URI of the VM object:
         $SessionId = $global:DefaultCIServers.SessionId | Where-Object { $VMObj.href -match $_.ServiceUri }

        try {
             $vmxml = Invoke-RestMethod -Method Get -Uri "$($VMObj.href)/virtualHardwareSection/disks" -Headers @{'x-vcloud-authorization'=$SessionId; 'Accept'="application/*+xml;version=$($vCDAPIVersion)"} -ErrorAction Stop
         } catch {
             Write-Host -ForegroundColor Red "Error attempting to get VM details from API:"
             Write-Host -ForegroundColor Red "Status Code: $($_.Exception.Response.StatusCode.value__)"
             Write-Host -ForegroundColor Red "Status Description: $($_.Exception.Response.StatusDescription)"
             Break
         }

        # Build an empty object for the VM disk details:
         $vmdisks = @()

        # RASD resource type 17 is a hard disk attached to a VM:
         foreach($disk in ($vmxml.RasdItemsList.Item | Where-Object -Property ResourceType -eq 17)) {
             
             # Dereference the StorageProfileHref for each disk to get the Storage Profile Name:
             try {
                 $sprof = Invoke-RestMethod -Method Get -Uri "$($disk.HostResource.storageProfileHref)" -Headers @{'x-vcloud-authorization'=$SessionId; 'Accept'="application/*+xml;version=$($vCDAPIVersion)"} -ErrorAction Stop
             } catch {
                 Write-Host -ForegroundColor Red "Error attempting to get Storage Profile Name:"
                 Write-Host -ForegroundColor Red "Status Code: $($_.Exception.Response.StatusCode.value__)"
                 Write-Host -ForegroundColor Red "Status Description: $($_.Exception.Response.StatusDescription)"
                 Break
             }
             
             $diskprops = @{
                 VMName         = [string]$VMObj.Name
                 InstanceID     = [string]$disk.InstanceID
                 StorageProfile = [string]$sprof.VdcStorageProfile.Name
                 CapacityGB     = [float][math]::Round(($disk.VirtualQuantity / 1024 / 1024 / 1024),3)
                 ElementName    = [string]$disk.ElementName
             }

            $diskobj = New-Object PSObject -Property $diskprops
             $vmdisks += $diskobj
         }

        return $vmdisks
     } # end process

}
Export-ModuleMember -Function Get-CIVMStorageDetail</pre>

And here is example output from the script for our test VM:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/09/image_thumb-3.png" alt="image" width="414" height="272" border="0" />][4]

Hope this is useful to some of you and as always, appreciate any comments/feedback.

I’d also love to know if there’s an easier way of generating this information.

Jon.

 [1]: https://kiwicloud.ninja/wp-content/uploads/2018/09/image.png
 [2]: https://kiwicloud.ninja/wp-content/uploads/2018/09/image-1.png
 [3]: https://kiwicloud.ninja/wp-content/uploads/2018/09/image-2.png
 [4]: https://kiwicloud.ninja/wp-content/uploads/2018/09/image-3.png