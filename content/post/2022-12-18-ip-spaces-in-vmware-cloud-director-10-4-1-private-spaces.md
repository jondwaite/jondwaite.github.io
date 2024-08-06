---
title: IP Spaces in VMware Cloud Director 10.4.1 – Part 2 – Private IP Spaces
author: Jon Waite
type: post
date: 2022-12-18T10:01:10+00:00
url: /2022/12/ip-spaces-in-vmware-cloud-director-10-4-1-private-spaces/
categories:
  - IP Spaces
  - vCloud Director
  - VMware
tags:
  - vCloud Director
  - VMware

---
In an <a href="https://kiwicloud.ninja/?p=69005" data-type="post" data-id="69005">earlier</a> post I wrote about the new IP Spaces feature in VMware Cloud Director (VCD) release 10.4.1 and the ability to create public IP spaces shared by multiple tenants. In this post I&#8217;ll look at the ability to create private IP spaces within a tenant organization and some of the considerations for how these can be used. There is also a <a href="https://kiwicloud.ninja/?p=69044" data-type="post" data-id="69044">part 3</a> of this series that looks at the tenant experience, compatibility and has a summary of IP Spaces.

As a tenant administrator, IP Spaces are visible under the Networking and then IP Spaces options (if enabled by our provider):<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="170" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-14-800x170.png" alt="" class="wp-image-69029" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-14-800x170.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-14-300x64.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-14-768x163.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-14-1536x326.png 1536w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-14-150x32.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-14-250x53.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-14.png 1861w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Clicking NEW starts the configuration workflow, the first screen allows us to provide a name for the new IP Space and an optional description:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="267" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-15-800x267.png" alt="" class="wp-image-69030" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-15-800x267.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-15-300x100.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-15-768x256.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-15-150x50.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-15-250x83.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-15.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Clicking NEXT prompts us to chose whether route adverts will be globally enabled or not for this IP Space, note the warning that per-network advertisement must still be enabled for each network that requires this even if this setting is globally enabled:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="260" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-16-800x260.png" alt="" class="wp-image-69031" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-16-800x260.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-16-300x97.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-16-768x249.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-16-150x49.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-16-250x81.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-16.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Clicking NEXT allows us to specify the overall IP space being defined, and the subset of this space which we are adding to this VCD environment. The ADD button can be used to add multiple network definitions to the internal scope if required. In this case our example tenant organization is using 10.0.0.0/8 globally for their internal networks, and a subset of this range of 10.1.0.0/16 will be used within the VCD environment:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="372" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-17-800x372.png" alt="" class="wp-image-69032" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-17-800x372.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-17-300x139.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-17-768x357.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-17-150x70.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-17-250x116.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-17.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Next we can assign a pool of IP addresses which can be allocated individually (e.g. for Load Balancer IP addresses used in the NSX Advanced Load Balancer). In this example I&#8217;ve reserved the upper half of the 10.1.0.0/16 address space for use in this way:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="256" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-18-800x256.png" alt="" class="wp-image-69036" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-18-800x256.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-18-300x96.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-18-768x246.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-18-150x48.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-18-250x80.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-18.png 1155w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Now we can assign the lower part of the pool which can be allocated as contiguous blocks (e.g. for OrgVDC networks within our tenancy), we specify the block size and the number of blocks to be created &#8211; in this example we chose a /24 block size and 128 total blocks, the down arrow allows us to preview the sequence of blocks that will be created for us:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="248" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-19-800x248.png" alt="" class="wp-image-69037" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-19-800x248.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-19-300x93.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-19-768x238.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-19-150x46.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-19-250x77.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-19.png 1155w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Finally we get to review our choices and make sure that all the assignments look correct:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="682" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-20-800x682.png" alt="" class="wp-image-69038" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-20-800x682.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-20-300x256.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-20-768x654.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-20-150x128.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-20-176x150.png 176w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-20.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

That completes the configuration of a new private IP space for our tenant organization, this can now be used when creating IP assignments within VCD, in the <a href="https://kiwicloud.ninja/?p=69044" data-type="post" data-id="69044">final part of this series</a> I show how a tenant can easily consume from (from either public or private IP Spaces) and use these for their networking requirements.

As always, comments and feedback welcome via here or twitter ([@jondwaite][1])

Jon.

 [1]: https://twitter.com/jondwaite