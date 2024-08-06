---
title: Working with vCloud Metadata in PowerCLI – Part 2
author: Jon Waite
type: post
date: 2016-03-05T23:55:21+00:00
url: /2016/03/working-with-vcloud-metadata-in-powercli-part-2/
categories:
  - PowerShell
  - VMware
tags:
  - Examples
  - Metadata
  - PowerCLI
  - vCloud Director 8
  - VMware

---
In [part 1][1] of this series I posted some code for a PowerShell module to manipulate (view/add/delete) metadata tags on vCloud Director objects.

As promised, in this followup post I&#8217;ve given some examples of syntax and possible use-cases for the module.

The PowerShell Import-Module cmdlet can be used to add my module to your PowerCLI environment:

`Import-Module "C:\PowerShell\CIMetadata.psm1"`

(assuming you&#8217;ve saved the code from part 1 as C:\PowerShell\CIMetadata.psm1). This can also be added to your PowerShell profile so that the cmdlets are always available when you start a new PowerShell session.

First we need to connect to a vCloud environment using Connect-CIServer:

`Connect-CIServer -Server '<URI to my cloud provider>' -Org '<My organisation>'`

Once connected we can get a list of the vApps in our environment:

<img loading="lazy" decoding="async" class="aligncenter wp-image-37 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta01.png" alt="cloudmeta01" width="1407" height="215" srcset="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta01.png 1407w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta01-300x46.png 300w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta01-768x117.png 768w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta01-1024x156.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta01-250x38.png 250w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta01-150x23.png 150w" sizes="(max-width: 1407px) 100vw, 1407px" /> So in this example organisation we have a single vApp with two VMs (server01 and server02) in it. Now lets see what existing metadata exists for them:

<img loading="lazy" decoding="async" class="aligncenter wp-image-51 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta02.png" alt="cloudmeta02" width="1438" height="196" srcset="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta02.png 1438w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta02-300x41.png 300w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta02-768x105.png 768w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta02-1024x140.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta02-250x34.png 250w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta02-150x20.png 150w" sizes="(max-width: 1438px) 100vw, 1438px" />  So we have some useful-ish metadata on the vApp itself, but nothing for the individual server VMs. Let&#8217;s say we have a script that runs (or we can call) after our VM backups run that we want to use to provide feedback on when our individual VMs were last backed up. We could do this by adding a &#8216;LastBackup&#8217; key to the VM with a Date/Time value of &#8216;Now&#8217; at the time the backup runs and a boolean value of &#8216;True&#8217; to a key &#8216;LastBackupSuccess&#8217; to indicate this was successful. Note that we&#8217;re using the special value &#8216;Now&#8217; for a Date/Time value to indicate the current date/time.

<img loading="lazy" decoding="async" class="aligncenter wp-image-53 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta03.png" alt="cloudmeta03" width="1144" height="327" srcset="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta03.png 1144w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta03-300x86.png 300w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta03-768x220.png 768w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta03-1024x293.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta03-250x71.png 250w, https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta03-150x43.png 150w" sizes="(max-width: 1144px) 100vw, 1144px" /> 

If we now check the VM metadata we can see that these keys/values have been added, both from PowerShell:

<img loading="lazy" decoding="async" class="aligncenter wp-image-43 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta04.png" alt="cloudmeta04" width="1435" height="107" /> and from the vCloud Director web UI:

<img loading="lazy" decoding="async" class="aligncenter wp-image-44 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta05.png" alt="cloudmeta05" width="981" height="439" /> It&#8217;s much more likely in a service provider context that these fields would actually be updated by our service provider using a &#8216;system&#8217; context user account and using the &#8216;-Visibility&#8217; option to make these key/values read-only to a tenant user. This could be accomplished using the following (connected as a system user to the vCloud environment):

<img loading="lazy" decoding="async" class="aligncenter wp-image-45 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta06.png" alt="cloudmeta06" width="1441" height="232" /> The vCD UI now shows (from a tenant perspective) the values we&#8217;ve entered as read-only/non-editable:

<img loading="lazy" decoding="async" class="aligncenter wp-image-46 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta07.png" alt="cloudmeta07" width="981" height="491" /> 

The fields returned by Get-CIMetaData can also be used in PowerShell scripts to filter results. For example, if our service provider uses the key &#8216;SPLastBackupSuccess&#8217; to indicate backup success/failure and has set &#8216;server02&#8217; in our vApp as having failed it&#8217;s backup we can (as a tenant) use the following to see any VMs in our vApp which have failed their backup:

<img loading="lazy" decoding="async" class="aligncenter wp-image-47 size-full" src="https://kiwicloud.ninja/wp-content/uploads/2016/03/cloudmeta08.png" alt="cloudmeta08" width="1442" height="111" /> 

These are just a couple of examples of how manipulating vCloud Director metadata from PowerShell could be useful, I&#8217;m sure there are many other use-cases out there too &#8211; e.g. indicating replication status to a remote cloud/data center or maintaining a reference link between another system and vCloud VMs. Don&#8217;t forget that you can also attach metadata to most vCloud Director objects in addition to vApps and VMs.

Let me know in the comments if you&#8217;ve found this useful and if there&#8217;s any other use-cases or scenarios you&#8217;d like to see.

Jon.

 [1]: http://152.67.105.113/2016/02/working-with-vcloud-metadata-in-powercli-part-1/