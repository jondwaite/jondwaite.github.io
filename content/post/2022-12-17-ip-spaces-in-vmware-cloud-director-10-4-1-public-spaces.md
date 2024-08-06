---
title: 'IP Spaces in VMware Cloud Director 10.4.1 – Part 1 – Introduction & Public IP Spaces'
author: Jon Waite
type: post
date: 2022-12-17T09:06:05+00:00
url: /2022/12/ip-spaces-in-vmware-cloud-director-10-4-1-public-spaces/
categories:
  - IP Spaces
  - vCloud Director
  - VMware
tags:
  - vCloud Director
  - VMware

---
VMware recently <a rel="noreferrer noopener" href="https://docs.vmware.com/en/VMware-Cloud-Director/10.4.1/rn/vmware-cloud-director-1041-release-notes/index.html" target="_blank">released</a> version 10.4.1 of VMware Cloud Director (VCD) &#8211; VMware add a new feature known as IP Spaces. IP Spaces allow providers and tenant customers to manage IP address allocations within VCD.

What was originally meant to be a &#8216;quick look&#8217; at IP Spaces and the capabilities they add has somehow turned into a 3-part series, so please follow the links below to the bit that interests you most.

<ul class="wp-block-list">
  <li>
    Part 1 &#8211; Introduction & Public IP Spaces (this post)
  </li>
  <li>
    <a href="https://kiwicloud.ninja/?p=69028" data-type="post" data-id="69028">Part 2 &#8211; Private IP Spaces</a>
  </li>
  <li>
    <a href="https://kiwicloud.ninja/?p=69044" data-type="post" data-id="69044">Part 3 &#8211; IP Spaces Tenant Experience, Compatibility and Summary</a>
  </li>
</ul>

In this first post we&#8217;ll look at the first of three possible types of IP Spaces, &#8216;Public&#8217; IP Spaces and see how these can be configured by the Service Provider to provide extra capability and features not previously available in Cloud Director. In particular, Service Providers can now create a shared pool of IP addresses which can be &#8216;drawn down&#8217; on by tenant organisations (within a quota limit which can be assigned both globally, and overridden for individual tenants).

This ability to have a &#8216;floating pool&#8217; of available addressing which can be shared and reused by multiple tenants (rather than having to statically assign and manage each individual address to a specific tenant) is a welcome improvement, and in partular with the scarcity of IPv4 address space will make it much easier for Service Providers to manage and provide flexible public IP address space allocations to their customers.

IP Spaces can be found in the VCD provider UI under the Resources / Cloud Resources tabs:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="481" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-800x481.png" alt="" class="wp-image-69007" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-800x481.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-300x180.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-768x462.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-1536x923.png 1536w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-150x90.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-250x150.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image.png 1549w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

To create a new public IP Space (for example to provide internet addresses), click the NEW text which opens the IP Space creation workflow:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="485" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-1-800x485.png" alt="" class="wp-image-69008" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-1-800x485.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-1-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-1-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-1-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-1-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-1.png 1157w" sizes="(max-width: 800px) 100vw, 800px" /> <figcaption class="wp-element-caption">In the next step you name your new IP Space and (optionally) provide a description:</figcaption></figure> <figure class="wp-block-image size-large"><img loading="lazy" decoding="async" width="800" height="485" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-2-800x485.png" alt="" class="wp-image-69009" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-2-800x485.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-2-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-2-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-2-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-2-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-2.png 1157w" sizes="(max-width: 800px) 100vw, 800px" /><figcaption class="wp-element-caption">Next you decide whether to enable route advertisement for allocations of blocks of IP addresses (referred to as &#8216;Prefixes&#8217; in the UI), for this example we won&#8217;t:</figcaption></figure> <figure class="wp-block-image size-large"><img loading="lazy" decoding="async" width="800" height="485" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-3-800x485.png" alt="" class="wp-image-69010" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-3-800x485.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-3-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-3-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-3-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-3-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-3.png 1157w" sizes="(max-width: 800px) 100vw, 800px" /></figure> 

In this example, we&#8217;re going to assume that our &#8216;public&#8217; assigned internet address block is 10.10.10.0/24 (255 addresses from 10.10.10.1 through 10.10.10.255), and:

<ul class="wp-block-list">
  <li>
    We want to assign the block from 10.10.10.128 through 10.10.10.191 (64 addresses) to this instance of VCD for our tenants to use.
  </li>
  <li>
    We want to allow the first 32 addresses (10.10.10.128/27 or 10.10.10.128-10.10.10.159) to be assigned individually.
  </li>
  <li>
    We also would like the 2nd block of 32 addresses (10.10.10.160/27 or 10.10.10.160-10.10.10.191) to be requestable in blocks of 4 contiguous addresses by tenants.
  </li>
  <li>
    Finally, we only want to allow each tenant by default to assign up to 2 individual IP addresses from the first block and 1 segment from the 2nd block.
  </li>
</ul>

Here&#8217;s how we can configure these settings in the VCD UI, first we define our Scope (including the &#8216;superscope&#8217; from which the /26 we want is derived:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="485" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-4-800x485.png" alt="" class="wp-image-69012" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-4-800x485.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-4-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-4-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-4-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-4-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-4.png 1157w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Next we add the IP Range from which tenants will be able to request individual IP addresses (in our example the /27 gives us addresses 10.10.10.128 to 10.10.10.159 inclusive):<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="485" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-5-800x485.png" alt="" class="wp-image-69013" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-5-800x485.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-5-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-5-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-5-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-5-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-5.png 1157w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Next we add the IP Prefix blocks that we want to be able to issue, we can use the arrow to the left side to verify that the ranges being created match the blocks that we desire. We use /30 on our range to create blocks of 4 addresses for each prefix and then specify we want to create 8 of these blocks for a total of 32 addresses:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="485" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-6-800x485.png" alt="" class="wp-image-69014" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-6-800x485.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-6-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-6-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-6-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-6-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-6.png 1157w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Next we can set the default quota limits which will apply to all tenant Organizations that we don&#8217;t explicity set alternative limits for (or leave these as &#8216;Unlimited&#8217;):<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="485" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-7-800x485.png" alt="" class="wp-image-69015" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-7-800x485.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-7-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-7-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-7-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-7-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-7.png 1157w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Finally we get a summary review panel and can check and accept everything to confirm creation of the new public IP space:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="485" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-8-800x485.png" alt="" class="wp-image-69016" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-8-800x485.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-8-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-8-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-8-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-8-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-8.png 1157w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

The IP Spaces view shows our newly created pool with a summary of how much address space is in use from it:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="481" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-9-800x481.png" alt="" class="wp-image-69017" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-9-800x481.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-9-300x180.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-9-768x462.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-9-1536x924.png 1536w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-9-150x90.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-9-250x150.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-9.png 1550w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

To make this new IP space available to tenants, we need to assign it to the Provider Gateway supporting the tenant Tier 1 gateways:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="481" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-10-800x481.png" alt="" class="wp-image-69018" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-10-800x481.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-10-300x180.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-10-768x462.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-10-1536x924.png 1536w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-10-150x90.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-10-250x150.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-10.png 1550w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Clicking NEW initiates a new workflow which allows us to specify the Tenant-facing name for the pool (an extremely welcome feature), for example:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="486" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-11-800x486.png" alt="" class="wp-image-69019" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-11-800x486.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-11-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-11-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-11-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-11-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-11.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

In the next page we select our newly create IP Space, note the warning that the IP Space reference cannot be altered once assigned to the Uplink<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="486" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-12-800x486.png" alt="" class="wp-image-69021" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-12-800x486.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-12-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-12-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-12-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-12-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-12.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Clicking next shows a summary page from which the details can be confirmed with a Finish button to confirm the configuration choices:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="486" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-13-800x486.png" alt="" class="wp-image-69022" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-13-800x486.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-13-300x182.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-13-768x466.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-13-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-13-247x150.png 247w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-13.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

And that&#8217;s it! The IP Space is now fully configured and can be used by tenants to request public IP addresses (or Prefix blocks of addresses if allowed) for use in their environments. Organizational overrides can be added to limit or extend the range of addresses that can be requested by each tenant.

<a href="https://kiwicloud.ninja/?p=69028" data-type="post" data-id="69028">Part 2</a> of this series considers using Private IP Spaces as a tenant to manage local IP address space used within VCD.

Hopefully this has given you a good feel for how the Public IP Space operates in VCD 10.4.1 and can be extremely flexibly used to provide individual (or blocks) of IP addresses to tenants.

As always, comments and feedback welcome via here or twitter ([@jondwaite][1])

Jon.

 [1]: https://twitter.com/jondwaite