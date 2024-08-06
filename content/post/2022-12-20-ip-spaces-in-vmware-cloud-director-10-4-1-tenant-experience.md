---
title: 'IP Spaces in VMware Cloud Director 10.4.1 – Part 3 – Tenant Experience, Compatibility & Summary'
author: Jon Waite
type: post
date: 2022-12-20T00:08:20+00:00
url: /2022/12/ip-spaces-in-vmware-cloud-director-10-4-1-tenant-experience/
categories:
  - IP Spaces
  - vCloud Director
  - VMware
tags:
  - 10.4.1
  - IP Spaces
  - vCloud Director
  - VMware

---
In <a href="https://kiwicloud.ninja/?p=69005" data-type="post" data-id="69005">part 1</a> of this series I looked at the process for a Service Provider to allocate a public block of IP addresses to be consumable by VMware Cloud Director (VCD) tenant organizations.

In <a href="https://kiwicloud.ninja/?p=69028" data-type="post" data-id="69028">part 2</a> of this series I showed the process by which a Service Provider or tenant organization can allocate IP Spaces for consumption within their environment.

This 3rd part will show how this all appears from the tenant perspective within VCD and how they can request and assign addressing from IP Spaces in their environment.

I&#8217;ll be using a single VCD tenant &#8216;Tyrell Corporation&#8217; in this example who has previously had both a public IP space assigned to them and a private IP space created. I&#8217;ll cover the following in this post:

<ul class="wp-block-list">
  <li>
    <a href="#float-public" data-type="URL">Requesting and using a new &#8216;floating&#8217; public IP address</a>
  </li>
  <li>
    <a href="#float-private" data-type="internal" data-id="#float-private">Requesting and using a new &#8216;floating&#8217; private IP address</a>
  </li>
  <li>
    <a href="#prefix-private" data-type="internal" data-id="#prefix-private">Requesting and using a new &#8216;prefix&#8217; private IP address range</a>
  </li>
  <li>
    <a href="#ipspaces-compat">Notes on IP Spaces Compatibility (as of December 2022)</a>
  </li>
  <li>
    <a href="#summary">Summary and final thoughts</a>
  </li>
</ul>

### Requesting and using a new &#8216;floating&#8217; public IP Address {#float-public.wp-block-heading}

Logged in to VCD as our tenant administrator and going to Networking / IP Spaces, we can see that two IP Spaces are available to us &#8211; a &#8216;Public IP Pool&#8217; and &#8216;Tyrell Internal&#8217; private pool:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="222" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-22-800x222.png" alt="" class="wp-image-69050" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-22-800x222.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-22-300x83.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-22-768x213.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-22-150x42.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-22-250x69.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-22.png 1285w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Selecting the &#8216;Public IP Pool&#8217; we get a summary that shows we already have 1 Floating IP allocated which is in use (in this case this is the &#8216;outside&#8217; address used on our Orgs Edge Gateway):<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="451" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-23-800x451.png" alt="" class="wp-image-69051" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-23-800x451.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-23-300x169.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-23-768x433.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-23-150x85.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-23-250x141.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-23.png 1285w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

To allocate an additional Floating &#8216;Public&#8217; IP address we go to &#8216;Floating IPs&#8217; and click the REQUEST button:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="205" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-24-800x205.png" alt="" class="wp-image-69052" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-24-800x205.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-24-300x77.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-24-768x197.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-24-150x39.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-24-250x64.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-24.png 1285w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

The pop-up dialog asks how many floating IPs we want to request (up to 5 can be requested at a time):

<div class="wp-block-image">
  <figure class="aligncenter size-full"><img loading="lazy" decoding="async" width="580" height="214" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-25.png" alt="" class="wp-image-69053" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-25.png 580w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-25-300x111.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-25-150x55.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-25-250x92.png 250w" sizes="(max-width: 580px) 100vw, 580px" /></figure>
</div>

Once the request is made, we may be told we have reached our quota and that no more addresses are available to us (in which case the Service Provider will need to allocate additional addresses by either changing the global quota or adding an override quota for our Org). In this case, we have available quota and the additional Public IP address is assigned to our Org and is ready for assignment:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="220" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-26-800x220.png" alt="" class="wp-image-69054" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-26-800x220.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-26-300x82.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-26-768x211.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-26-150x41.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-26-250x69.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-26.png 1284w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Selecting the newly allocated address allows us to flag the address as &#8216;Manual&#8217; if we wish to use it for something outside of what our VCD environment is aware of, as well as release the IP back to the provider if we no longer need it:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="220" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-27-800x220.png" alt="" class="wp-image-69055" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-27-800x220.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-27-300x82.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-27-768x211.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-27-150x41.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-27-250x69.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-27.png 1284w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

Once we have the IP allocated, we can assign it to services on our Edge Gateway (for example, as a new DNAT external address for a service we wish to publish to the Internet), or assign to a NSX Advanced Load Balancer as the Load Balancer address.

Note that VCD won&#8217;t let IP addresses flagged as &#8216;Used&#8217; be released back to the provider pool so the services referencing an address must be reconfigured first.

### Requesting and using a new &#8216;floating&#8217; private IP Address {#float-private.wp-block-heading}

The process to request a new private floating IP address is similar to the process for a public IP address, although typically there won&#8217;t be any quota on available private addresses. We first select our private IP space and can see the summary for this:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="645" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-28-800x645.png" alt="" class="wp-image-69056" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-28-800x645.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-28-300x242.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-28-768x619.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-28-150x121.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-28-186x150.png 186w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-28.png 1285w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

In this case we&#8217;ve defined our range as 192.168.0.0/16 from which 192.168.0.0/20 will be used within our VCD environment. We&#8217;ve then further subdivided 192.168.0.0/20 into a lower block of 8 prefix ranges (192.168.0.0/24, 192.168.1.0/24 etc.) and a pool of floating addresses from 192.168.8.1 to 192.168.15.22.

To request a new Floating IP address we select the Floating IPs item from the left panel:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="211" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-29-800x211.png" alt="" class="wp-image-69057" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-29-800x211.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-29-300x79.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-29-768x203.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-29-150x40.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-29-250x66.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-29.png 1285w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

We can see that a single IP address has already been allocated and is in use by a Load Balancer instance, to allocate additional floating IPs we select ALLOCATE and provide the number of addresses required between 1 and 5:

<div class="wp-block-image">
  <figure class="aligncenter size-full"><img loading="lazy" decoding="async" width="580" height="214" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-30.png" alt="" class="wp-image-69058" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-30.png 580w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-30-300x111.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-30-150x55.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-30-250x92.png 250w" sizes="(max-width: 580px) 100vw, 580px" /></figure>
</div>

Our new Floating IP address is allocated and shown to us with a type of &#8216;Unused&#8217;, as with public addresses we have the option to flag this address as being &#8216;Manual Use&#8217; (outside of VCDs awareness):<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="228" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-31-800x228.png" alt="" class="wp-image-69059" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-31-800x228.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-31-300x86.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-31-768x219.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-31-150x43.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-31-250x71.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-31.png 1282w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

We also have the option to release the IP address back to the pool if no longer required (and not currently in use). Once obtained we can now use the floating IP that has been allocated to us &#8211; typically this would be assigned to an NSX Advanced Load Balancer for use to internally load balance a service which our internal users consume (as can be seen for the existing 192.168.8.1 IP address).

### Requesting and using a new &#8216;prefix&#8217; private IP Address Range {#prefix-private.wp-block-heading}

When requesting a new &#8216;prefix&#8217; IP address block, this can be done either by first allocating the block in IP Spaces and then assigning this to a new network, or directly at the time the new network is created. We&#8217;ll look at the 2nd option here as the first behaves very similarly to requesting a floating private IP address.

From the tenant Networking / Networks tab, select NEW to create a new network for the OrgVDC:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="173" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-32-800x173.png" alt="" class="wp-image-69061" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-32-800x173.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-32-300x65.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-32-768x166.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-32-150x32.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-32-250x54.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-32.png 1285w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

This opens the New Organization VDC Network workflow, where we select our VDC and click Next:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="471" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-33-800x471.png" alt="" class="wp-image-69062" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-33-800x471.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-33-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-33-768x452.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-33-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-33-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-33.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

We can chose whether this new network will be Routed via our Edge gateway or Isolated, in this example I&#8217;ll just create an Isolated network:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="471" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-34-800x471.png" alt="" class="wp-image-69063" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-34-800x471.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-34-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-34-768x452.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-34-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-34-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-34.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

In the General step, we give the network a name and network details &#8211; if we select the dropdown arrow next to the Gateway CIDR field we see any accessible private networks available to us and can then chose a pre-allocated prefix network (if available), or ask to request a new one if not:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="471" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-35-800x471.png" alt="" class="wp-image-69064" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-35-800x471.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-35-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-35-768x452.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-35-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-35-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-35.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

As we have no pre-allocated range available (&#8216;None&#8217;) in this example, we&#8217;ll use &#8216;request&#8217;, after a short request phase, we get assigned a new prefix network from our IP Spaces definition and are told what this will be:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="471" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-37-800x471.png" alt="" class="wp-image-69066" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-37-800x471.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-37-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-37-768x452.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-37-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-37-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-37.png 1156w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

The rest of the network creation workflow functions as previously using the &#8216;legacy&#8217; network model so there is no need to go through the rest of these steps.

Note that IP Spaces will not permit the same network allocation (IP addresses or prefixes) to be used more than once within an OrgVDC. This can potentially break some use-cases.

For example, as a tenant I may want to allocate a network to do DR testing which shares the same IP address space as my Production server network (although being disconnected from it &#8211; possibly in an isolated network). IP Spaces will not currently allow me to define a network for DR which conflicts/overlaps with my production address space. I&#8217;m not sure what VMware are intending to be used in this situation, but since VCD 10.4.1 is the first release of this capability I&#8217;m sure we will see consideration of these types of requirements in future revisions.

### IP Spaces Compatibility (as at Dec 2022) {#ipspaces-compat.wp-block-heading}

While testing out IP Spaces I also noticed incompatibilities with VMware Container Services Extension (CSE) 4.0 for Cloud Director. The core of the issue appears to be that CSE doesn&#8217;t know how to request IP address allocations from IP Spaces. Annoyingly right now this means that any CSE cluster deployment will fail in an IP Spaces environment unless:

<ul class="wp-block-list">
  <li>
    An IP address has been requested (but not allocated) from an accessible IP Space (private or public)
  </li>
  <li>
    The obtained IP address is manually specified in the &#8216;Control Plan IP (Optional)&#8217; field when deploying a cluster:
  </li>
</ul><figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="469" src="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-38-800x469.png" alt="" class="wp-image-69068" srcset="https://kiwicloud.ninja/wp-content/uploads/2022/12/image-38-800x469.png 800w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-38-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-38-768x451.png 768w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-38-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-38-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2022/12/image-38.png 1154w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

This will allow the CSE cluster to deploy successfully, but attempts to provision new Kubernetes resources that use the LoadBalancer ingress type into the cluster will fail as CSE&#8217;s Cloud Director integration doesn&#8217;t understand how to request additional service IP addresses from IP Spaces.

This can probably be worked around by manually specifying ingress addresses for each deployed service, but is not really a graceful tenant experience. I&#8217;m sure that the VMware teams involved with CSE and VCD will come up with a way to fix this in subsequent versions.

### Summary {#summary.wp-block-heading}

When I started this post I had no idea it would turn into a 3-part series, or of some of the issues I would experience while testing IP Spaces. I have to say the move towards pooled address management for public addressing is a potentially huge benefit for Service Providers who are increasingly having to juggle the limited resource of available public IP addresses.

I also like the move to provide tenants with more capability over requesting (and releasing) their consumption of public IP addresses along with managing the use of IP address spaces within their environments.

If the wrinkles around how overlapping ranges can be handled more gracefully, and the interactions for other services such as Container Service Extension improved then I can see IP Spaces as being useful for most Service Providers in future.

One other question remains about how VMware will help Service Providers migrate existing environments using the &#8216;legacy&#8217; IP allocation methods to and from IP Spaces as this is a significant task to perform manually and automation could really help to make this easier.

In short &#8211; IP Spaces looks really promising and has a lot of capability in its first release with VCD 10.4.1, if VMware can keep developing it and iron out the wrinkles that current exist it will become a very welcome enhacement to VCD capabilities for both Service Providers and tenants.

As always, I hope you&#8217;ve found this series of posts useful and comments/feedback welcome in here or [@jondwaite][1] on Twitter.

Jon.

 [1]: https://twitter.com/jondwaite