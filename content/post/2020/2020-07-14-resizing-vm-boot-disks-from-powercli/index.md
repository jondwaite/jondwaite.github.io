---
title: Resizing VM boot disks from PowerCLI
author: Jon Waite
type: post
date: 2020-07-13T22:13:13+00:00
excerpt: PowerShell function to resize first hard disk in VMware Cloud Director virtual machines.
url: /2020/07/resizing-vm-boot-disks-from-powercli/
codeLineNumbers: true
categories:
  - PowerShell
  - vCloud Director
tags:
  - PowerCLI
  - PowerShell
  - VMware Cloud Director

---
PowerShell can be a great way to administer Virtual Machines in VMware Cloud Director, but there are still some 'gaps' in the cmdlet cover for commonly required functions. As and when customers ask me about these I can usually find a reasonably easy way to fill the gap with some code to access the VCD API directly. This was the case in this post where a customer asked how they could easily resize the boot/first disk in a VCD VM. I was actually already working on a general solution for this which will be the subject of an upcoming blog post and PowerShell module, but in the meantime it was easiest to write a small function to deal with this specific use-case.

Note that this function will only update the disk device size, you will still need to update the partition table and extend the filesystem from within the guest OS to be able to take advantage of the new disk capacity. It can also only **increase** disk size as reducing disk size is both dangerous (risk of data loss) and not supported by the VCD API. In most modern Guest OS's (e.g. Windows, Linux) it is possible to rescan attached disks and extend partition size and filesystems without requiring the VM to be power-cycled.

The function I came up with is shown below, since there are a number of techniques involved here I have included commentary below as to what each part is doing which should make it easy for anyone wanting to write their own variation or similar functions.

```powershell
Function Update-CIVMBootDisk {
    Param(
        [Parameter(Mandatory=$true,ValueFromPipeline=$true)]$VM,
        [Parameter(Mandatory=$true)][int64]$NewSizeMB
    )

    $APIResponse = Invoke-RestMethod -Uri "$($Global:DefaultCIServers[0].ServiceUri)versions" `
        -Method 'Get' `
        -Headers @{'Accept'='application/*+xml'}
    $APIVersion = (($ApiResponse.SupportedVersions.VersionInfo | `
        Where-Object { $_.deprecated -eq $false }) | `
        Measure-Object -Property Version -Maximum).Maximum.ToString() + ".0"

    $SessionId = $global:DefaultCIServers[0].SessionId
    $Headers = @{'x-vcloud-authorization'=$SessionId;'Accept'="application/*+xml;version=$($APIVersion)"}
    $Uri = "$($VM.Href)/virtualHardwareSection/disks"
    $DiskSection = Invoke-RestMethod -Method 'Get' -Uri $Uri -Headers $Headers

    $disk = $DiskSection.RasdItemsList.Item | `
        Where-Object { (($_.AddressOnParent -eq 0) -and ($_.ResourceType -eq 17))}

    $disk.HostResource.capacity = $NewSizeMB
    $disk.VirtualQuantity = ($NewSizeMB * 1024 * 1024)
    $Headers += @{'Content-Type'='application/vnd.vmware.vcloud.rasdItemsList+xml'}

    try {
        $task = Invoke-RestMethod -Method 'Put' -Uri $Uri -Headers $Headers -Body ($DiskSection.InnerXml)
    }
    catch {
        Write-Error ("Exception returned attempting to modify VM disk size")    
        $_.Exception
        Exit
    }
    Write-Host ("Disk resize for VM $($VM.Name) submitted successfully.")
}
```

|Lines|Function|
|---|---|
|1-5|Standard Function definition, two inputs are required - the VM object which is being changed and the desired new size (in MB) of the first disk.|
|7-9|This is actually a single command split-across 3 lines for readability. It makes a call to the VCD API to determine the API versions supported. I've started using this in many of my scripts rather than 'hard-coding' an API version which then gets out of date as newer API versions are released and VMware deprecate older ones.|
|10-12|Processes the result returned from the previous line and determines the highest non-deprecated API version from the connected VCD API and stores this in $APIVersion for later use.|
|14|Determines the SessionId for the currently connected PowerCLI session (Connect-CIServer) which can then be used to authenticate our API requests in lines 17 and 27.|
|15|Builds the $Headers hash to be submitted with our API requests, the SessionId and APIVersion we determined earlier are used here.|
|16|Determines the API URI to be accessed based on the supplied VM HTTP reference.|
|17|Retrieves the XML representation of the virtualHardwareSection/disks section from the specified VM.|
|19-20|Finds the first hard disk attached to the VM (ResourceType 17 = Hard Disk)|
|22-23|Updates the disk size in the VM based on the new disk size supplied to the function. Note: Only the 'capacity' change on line 22 is strictly required since VCD appears to ignore the disk size specified in VirtualQuantity, but for consistency I update both values - since the VirtualQuantity is in bytes we need to multiply the MB size.|
|24|Add an additional value to the $Header hash to indicate the Content-Type of the document we will be submitting back to the API.|
|26-34|Attempts to update the API with the changed XML including the new disk size and return a meaningful error if this fails.|

#### Usage {.wp-block-heading}

To use the function, save it to a .PS1 file (e.g. 'Update-CIVMBootDisk.ps1') and then execute the file in your current PowerShell session. Once this is done you should be able to use the function against VMs in your current PowerCLI session.

#### Example {.wp-block-heading}

In the following example transcript the function is used to resize the boot disk of 'TestVM01' from 1GB to 2GB, the initial VM hard disk view looks like this:
![Disk resize - details before](diskresize-before-1.png)

Once connected to VCD (Connect-CIServer) the following can be used to resize (2048 since the size must be specified in MB):

```powershell
$VM = Get-CIVM -Name 'TestVM01'
$VM | Update-CIVMBootDisk -NewSizeMB 2048

Disk resize for VM TestVM01 submitted successfully.
```

Once the update has completed, refreshing the hard disk view now shows our first disk has been resized correctly:
![Disk resize - details after](diskresize-after.png)

I've tried to go into a bit more detail in how the script works than I usually have - hopefully this post will prove useful, as always comments and feedback are appreciated.

Jon.