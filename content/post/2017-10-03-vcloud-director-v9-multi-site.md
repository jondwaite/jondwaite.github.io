---
title: vCloud Director v9 Multi-site
author: Jon Waite
type: post
date: 2017-10-03T05:38:32+00:00
url: /2017/10/vcloud-director-v9-multi-site/
categories:
  - PowerShell
  - REST API
  - vCloud Director
  - VMware
tags:
  - Module
  - PowerCLI
  - PowerShell
  - REST API
  - vCloud Director
  - vCloud Director 9

---
Since vCloud Director v9 was released last week (and previously as part of the closed beta), one of the new features I&#8217;m most excited about is support for multi-site deployments. This allows vCloud Director environments for the first time to properly span federated sites (e.g. a tenant who has resources in multiple datacenter locations for resiliency/redundancy can now manage these in the same place).

Configuring multi-site support in vCloud Director v9 is a 2-part process:

1) The service provider has to configure federation between their vCloud Director instances.  
2) The tenant has to associate each of their Organizations in each vCloud Director instance.

This post will attempt to explain and show both processes, and there&#8217;s even a bonus of a PowerShell script I&#8217;ve written to help other service providers configure their site pairings. To help demonstrate the processes involved, I&#8217;ve built a test lab environment consisting of 4 separate vCloud Director instances (&#8216;Auckland&#8217;, &#8216;Wellington&#8217;, &#8216;Christchurch&#8217; and &#8216;Dunedin&#8217; sites). Each of these has it&#8217;s own vCloud Director, vCenter, NSX and ESXi hosts. I&#8217;ve also created a tenant organization (&#8216;Tenant X&#8217;) in all 4 instances and created and assigned a VDC to Tenant X in each location. Finally, I&#8217;ve federated Tenant X&#8217;s vCloud users with a directory service (Microsoft AD FS in this case) so that the same identity provider is available to all 4 vCloud instances.

If you&#8217;re not a service provider and just need to configure Organization pairing you&#8217;re probably safe to skip this section and proceed straight to the 2nd part of this post.

## Part 1 &#8211; Service Provider Site Pairing

The basic process for a service provider to pair sites is:

&#8211; Check (and configure if necessary) the vCloud site name in each location. Note by default in the initial vCloud Director 9 release this is simply a GUID string so you&#8217;ll probably want to change it to something more meaningful.  
&#8211; Download from each site the site association document (from /api/site/associations/localAssociationData)  
&#8211; Upload this site association document to each other site that you want to pair with (to /api/site/associations)

In a scenario with only 2 sites you need to perform this process twice (once in each direction), but for our example with 4 sites we need to do this a total of 12 times to pair every site with every other site.

Being a bit of a pain to do manually against the REST API interface I ran true to form and wrote a PowerShell module to simplify the process of both administering the site names and also pairing sites together. The module is available on my github repository at <a href="https://github.com/jondwaite/vCDSitePair" target="_blank" rel="noopener">https://github.com/jondwaite/vCDSitePair</a>. The script uses my <a href="http://152.67.105.113/2017/09/invoke-vcloud-powershell-module/" target="_blank" rel="noopener">Invoke-vCloud</a> module, so you&#8217;ll need that installed for it to run.

Once you&#8217;ve downloaded the vCDSitePair.psm1 file from github you can add it to your PowerShell session using &#8216;Import-Module &#8216;

There are a total of 4 functions provided by the module, and they are documented on the github repository but basically:

<table>
  <tr>
    <td>
      Get-vCloudSiteName
    </td>
    
    <td>
      Allows a service provider to check/confirm the &#8216;Site Name&#8217; assigned to a vCloud Director instance.
    </td>
  </tr>
  
  <tr>
    <td>
      Set-vCloudSiteName
    </td>
    
    <td>
      Allows a service provider to set/update the &#8216;Site Name&#8217; assigned to a vCloud Director instance.
    </td>
  </tr>
  
  <tr>
    <td>
      Get-vCloudSiteAssociations
    </td>
    
    <td>
      Shows the existing associations (if any) from a vCloud Director instance.
    </td>
  </tr>
  
  <tr>
    <td>
      Invoke-vCDPairSites
    </td>
    
    <td>
      Performs the 2-way exchange of localAssociationData documents to pair two vCloud Director instances.
    </td>
  </tr>
</table>

So to confirm/set the names of our &#8216;Auckland&#8217; (akl.mycloud.local) and &#8216;Christchurch&#8217; (chc.mycloud.local) sites we can use Get-vCloudSiteName and Set-vCloudSiteName:

<pre class="lang:ps decode:true">PS C:\&gt; Get-vCloudSiteName -siteDomain akl.mycloud.local
847f4785-f8a5-4e59-b0d7-5608723247dd

PS C:\&gt; Set-vCloudSiteName -siteDomain akl.mycloud.local -siteName 'A03 - Auckland DC'
Task submitted successfully, waiting for result
q=queued, P=pre-running, .=Task Running:
q
Task completed successfully
True

PS C:\&gt; Get-vCloudSiteName -siteDomain akl.mycloud.local
A03 - Auckland DC

Christchurch Site Name:

PS C:\&gt; Get-vCloudSiteName -siteDomain chc.mycloud.local
b34798aa-f722-4c06-a8a2-392f591cf450

PS C:\&gt; Set-vCloudSiteName -siteDomain chc.mycloud.local -siteName 'C00 - Christchurch DC'
Task submitted successfully, waiting for result
q=queued, P=pre-running, .=Task Running:
q 
Task completed successfully
True

PS C:\&gt; Get-vCloudSiteName -siteDomain chc.mycloud.local
C00 - Christchurch DC</pre>

Now we have set the site names, we can check if they are already associated:

&nbsp;

<pre class="lang:ps decode:true ">PS C:\&gt; Get-vCloudSiteAssociations -siteDomain akl.mycloud.local
Displaying site associations for site Id: urn:vcloud:site:847f4785-f8a5-4e59-b0d7-5608723247dd with site Name: A03 - Auckland DC
No site associations found

PS C:\&gt; Get-vCloudSiteAssociations -siteDomain chc.mycloud.local
Displaying site associations for site Id: urn:vcloud:site:b34798aa-f722-4c06-a8a2-392f591cf450 with site Name: C00 - Christchurch DC
No site associations found</pre>

Ok, so they&#8217;re not already associated, so we can run Invoke-vCDPairSites without the &#8216;WhatIf $false&#8217; to see what would happen:

<pre class="lang:ps decode:true ">PS C:\&gt; Invoke-vCDPairSites -siteAuri akl.mycloud.local -siteBuri chc.mycloud.local
Running in information mode only - no API changes will be made unless you run with -WhatIf $false
Site A returned site ID as: urn:vcloud:site:847f4785-f8a5-4e59-b0d7-5608723247dd
Site A returned site name as: A03 - Auckland DC
Site B returned site ID as: urn:vcloud:site:b34798aa-f722-4c06-a8a2-392f591cf450
Site B returned site name as: C00 - Christchurch DC
Not performing site association as running in information mode</pre>

That all looks good so now we can attempt the action pairing operation:

<pre class="lang:ps decode:true ">PS C:\&gt; Invoke-vCDPairSites -siteAuri akl.mycloud.local -siteBuri chc.mycloud.local -WhatIf $false
Running in implementation mode, API changes will be committed
Site A returned site ID as: urn:vcloud:site:847f4785-f8a5-4e59-b0d7-5608723247dd
Site A returned site name as: A03 - Auckland DC
Site B returned site ID as: urn:vcloud:site:b34798aa-f722-4c06-a8a2-392f591cf450
Site B returned site name as: C00 - Christchurch DC
Associating A03 - Auckland DC (Site A) with C00 - Christchurch DC (Site B)
Task submitted successfully, waiting for result
q=queued, P=pre-running, .=Task Running:
q 
Task completed successfully
Returned Result = True
Associating C00 - Christchurch DC (Site B) with A03 - Auckland DC (Site A)
Task submitted successfully, waiting for result
q=queued, P=pre-running, .=Task Running:
q 
Task completed successfully
Returned Result = True</pre>

And confirm the associations using Get-vCloudSiteAssociation again:

<pre class="lang:ps decode:true ">PS C:\&gt; Get-vCloudSiteAssociations -siteDomain akl.mycloud.local
Displaying site associations for site Id: urn:vcloud:site:847f4785-f8a5-4e59-b0d7-5608723247dd with site Name: A03 - Auckland DC
Associated sites:
https://chc.mycloud.local/api


href                    : https://akl.mycloud.local/api/site/associations/b34798aa-f722-4c06-a8a2-392f591cf450
type                    : application/vnd.vmware.admin.siteAssociation+xml
Link                    : {Link, Link, Link}
RestEndpoint            : https://chc.mycloud.local/api
TenantUiEndpoint        : https://chc.mycloud.local/tenant
RestEndpointCertificate : -----BEGIN CERTIFICATE-----
                          MII...
                          -----END CERTIFICATE-----
SiteId                  : urn:vcloud:site:b34798aa-f722-4c06-a8a2-392f591cf450
SiteName                : C00 - Christchurch DC
PublicKey               : -----BEGIN PUBLIC KEY-----
                          MII...
                          -----END PUBLIC KEY-----</pre>

(I&#8217;ve cut out the certificate dumps for brevity).

So now that our Auckland and Christchurch sites are paired we can move on with associating the Organization (&#8216;Tenant X&#8217;) between these sites. I&#8217;ve also been through and associated all of the other sites to each other, so by this stage &#8216;Auckland&#8217; is associated to &#8216;Wellington&#8217;,&#8217;Christchurch&#8217; and &#8216;Dunedin&#8217; etc.

## Part 2 &#8211; Organization Site Pairing

Originally I was intending to write PowerShell functions for this too, and while this is certainly possible, VMware have been nice to us and created the capability in the new vCloud Director tenant UI. Logging in as a user with &#8216;Organizational Administrator&#8217; access shows an &#8216;Administration&#8217; tab:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-193" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair01-1.png" alt="" width="1183" height="708" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair01-1.png 1183w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair01-1-300x180.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair01-1-768x460.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair01-1-1024x613.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair01-1-250x150.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair01-1-150x90.png 150w" sizes="(max-width: 1183px) 100vw, 1183px" /> 

Selecting the &#8216;Administration&#8217; tab reveals the site pairing options:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-194" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair02.png" alt="" width="1182" height="706" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair02.png 1182w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair02-300x179.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair02-768x459.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair02-1024x612.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair02-250x150.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair02-150x90.png 150w" sizes="(max-width: 1182px) 100vw, 1182px" /> 

When we select &#8216;Export Local Association Data&#8217; a file is downloaded (named &#8216;Download&#8217; weirdly enough) and this file can be uploaded to the &#8216;partner&#8217; site using the other &#8216;Create New Organization Association&#8217; button. Once completed, the association is shown in the panel &#8211; here is the Auckland site for &#8216;Tenant X&#8217; once the Christchurch Local Association Data has been uploaded to it:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-195" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair03.png" alt="" width="1287" height="689" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair03.png 1287w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair03-300x161.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair03-768x411.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair03-1024x548.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair03-250x134.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair03-150x80.png 150w" sizes="(max-width: 1287px) 100vw, 1287px" /> 

You can click on this panel to see the association details and even remove a site association if no longer required:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-196" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair04.png" alt="" width="1289" height="691" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair04.png 1289w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair04-300x161.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair04-768x412.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair04-1024x549.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair04-250x134.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair04-150x80.png 150w" sizes="(max-width: 1289px) 100vw, 1289px" /> 

Here&#8217;s the view of the &#8216;Christchurch&#8217; environment once I&#8217;ve paired the &#8216;other&#8217; 3 sites to it for this Organization:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-197" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair05.png" alt="" width="962" height="791" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair05.png 962w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair05-300x247.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair05-768x631.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair05-182x150.png 182w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair05-150x123.png 150w" sizes="(max-width: 962px) 100vw, 962px" /> 

Once we&#8217;ve added all our associations (and logged out and back in) we can see the new multi-site drop-down menu item which allows us to select from any of our datacenter locations:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-198" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair06.png" alt="" width="965" height="792" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair06.png 965w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair06-300x246.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair06-768x630.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair06-183x150.png 183w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair06-150x123.png 150w" sizes="(max-width: 965px) 100vw, 965px" /> 

And selecting one (&#8216;Auckland&#8217; in this case) takes us to the Auckland resource view:

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-199" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair07.png" alt="" width="965" height="795" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair07.png 965w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair07-300x247.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair07-768x633.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair07-182x150.png 182w, https://kiwicloud.ninja/wp-content/uploads/2017/10/orgpair07-150x124.png 150w" sizes="(max-width: 965px) 100vw, 965px" /> 

All in all, a little bit of a convoluted process, but at least it should only need to be done once and can then be left alone. Very excited to see what VMware do with this functionality in future &#8211; can definitely see a time when all of an Organization&#8217;s VMs are displayed / summarised in a single view regardless of which vCloud instance supports them.

I have several more thoughts generally on vCloud Director v9 which I&#8217;ll put into a separate post when I have time, but wanted to get this published for anyone else playing around with the new multi-site features.

As always, comments and feedback appreciated.

Jon.