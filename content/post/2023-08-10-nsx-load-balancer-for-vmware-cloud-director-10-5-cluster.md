---
title: NSX Load Balancer for VMware Cloud Director 10.5 Cluster
author: Jon Waite
type: post
date: 2023-08-10T10:04:47+00:00
url: /2023/08/nsx-load-balancer-for-vmware-cloud-director-10-5-cluster/
featured_image: https://kiwicloud.ninja/wp-content/uploads/2023/08/image-5-139x150.png
categories:
  - Cloud Director
  - NSX
tags:
  - Load Balancer
  - NSX
  - vCloud Director
  - VMware

---
Back in 2019, Tom Fojta wrote an <a rel="noreferrer noopener" href="https://fojta.wordpress.com/2019/06/28/load-balancing-vcloud-director-with-nsx-t/" target="_blank">excellent guide</a> on configuring NSX Load Balancer in front of a VMware Cloud Director (VCD) cluster. As I was in the process of rebuilding my home lab environment I wanted to try this out &#8211; eventually I&#8217;ll probably use the NSX Advanced Load Balancer (formerly known as AVI), but I don&#8217;t have that deployed yet, and &#8216;plain&#8217; NSX itself is more than capable of performing the task.

What I hadn&#8217;t considered was that the updates to supported cipher suites in the more recent releases of VCD mean that some extra configuration is now required to enable the load balancing and service monitoring work correctly.

The lab environment I&#8217;m currently rebuilding is a nested installation of VMware Cloud Foundation (VCF) 5 which includes vCenter 8, ESXi 8 and NSX 4.1 so these are the versions used here.

If we check the <a rel="noreferrer noopener" href="https://docs.vmware.com/en/VMware-Cloud-Director/10.5/rn/vmware-cloud-director-105-release-notes/" target="_blank">VCD 10.5 release notes</a> we see that only a reasonably small subset of security protocols and cipher suites are now supported &#8216;out of the box&#8217; by default:<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="800" height="163" src="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-800x163.png" alt="" class="wp-image-69097" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-800x163.png 800w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-300x61.png 300w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-768x156.png 768w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-150x31.png 150w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-250x51.png 250w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image.png 890w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

One option would be to follow the <a href="https://kb.vmware.com/s/article/88929" target="_blank" rel="noreferrer noopener">guidance in KB88929</a> as linked in the release notes to enable other combinations of protocol and ciphers, but I wanted to keep the VCD environment as unmodified as possible so I opted instead to allow for these in the configuration of the load balancer.

My goals for the configuration were:

<ul class="wp-block-list">
  <li>
    Terminate client SSL sessions on the Load Balancer so I didn&#8217;t have to worry about maintaining valid public certificates on each VCD cell server individually
  </li>
  <li>
    Service monitoring of the vCD cells so that a cell being down was correctly handled and the cell server removed from the pool until available again
  </li>
  <li>
    Support large request/response headers and body sizes as required by VCD
  </li>
  <li>
    Provide a platform against which I can automate the refresh/renewal of VCD public certificates via API calls to NSX and VCD rather than scripting of cell-management-tool (which I&#8217;m aiming to detail in a future post once complete)
  </li>
</ul>

I&#8217;m not going to go over the creation of a Load Balancer inside NSX in this article, as there are plenty of guides out there on how to configure this. Once I had a Tier-1 Gateway deployed to act as the host for my load balancer I first created a new SSL server profile &#8211; in NSX manager go to **Networking > Load Balancing > Profiles > Select Profile Type: SSL** and **Add SSL Profile**. I&#8217;ve shown the options I selected for supported protocols and ciphers in the screenshot below, I called this **vcd-profile**, you need to set the **SSL Suite** drop-down to **Custom** to get the individual cipher and protocol options to activate:

<div class="wp-block-image">
  <figure class="aligncenter size-large"><a href="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-1.png"><img loading="lazy" decoding="async" width="800" height="410" src="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-1-800x410.png" alt="" class="wp-image-69100" style="object-fit:cover" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-1-800x410.png 800w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-1-300x154.png 300w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-1-768x394.png 768w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-1-1536x787.png 1536w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-1-150x77.png 150w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-1-250x128.png 250w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-1.png 1723w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>

Once we&#8217;ve created an appropriate SSL profile, we also need to create an application profile to configure the required larger request and response headers and body sizes needed, I called my application profile **vcloud-https**. Note that we don&#8217;t need the API call from Tom&#8217;s original post any more as we can directly edit both request and response header size limits in the NSX UI, I&#8217;ve also set request body size to 50MB to allow ISO/OVA transfers:

<div class="wp-block-image">
  <figure class="aligncenter size-large"><a href="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-2.png"><img loading="lazy" decoding="async" width="800" height="418" src="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-2-800x418.png" alt="" class="wp-image-69107" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-2-800x418.png 800w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-2-300x157.png 300w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-2-768x401.png 768w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-2-150x78.png 150w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-2-250x131.png 250w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-2.png 1462w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>

Once the two profiles are created it&#8217;s reasonably straightforward to configure the load balancer as normal, first define the monitor, ensuring that you use the new **vcd-profile** for the **Server SSL Profile**, we can then use the **/cloud/server_status** monitoring endpoint and check for the cell server response **Service is up.**

<div class="wp-block-image">
  <figure class="aligncenter size-large"><a href="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-3.png"><img loading="lazy" decoding="async" width="800" height="400" src="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-3-800x400.png" alt="" class="wp-image-69110" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-3-800x400.png 800w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-3-300x150.png 300w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-3-768x384.png 768w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-3-1536x767.png 1536w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-3-150x75.png 150w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-3-250x125.png 250w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-3.png 1644w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>

Now we can define our server pool in the normal way and add our VCD cell server IPs to the pool. Next when creating our virtual server we create as L7 HTTP, specify our pool and make sure that our application profile is selected

<div class="wp-block-image">
  <figure class="aligncenter size-large"><a href="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-4.png"><img loading="lazy" decoding="async" width="800" height="303" src="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-4-800x303.png" alt="" class="wp-image-69115" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-4-800x303.png 800w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-4-300x114.png 300w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-4-768x291.png 768w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-4-150x57.png 150w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-4-250x95.png 250w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-4.png 1444w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>

Finally we can configure the server SSL Configuration from our virtual server definition and make sure that we enable SSL and use our **vcd-profile** when connecting via SSL to our backend cell servers:

<div class="wp-block-image">
  <figure class="aligncenter size-medium"><a href="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-6.png"><img loading="lazy" decoding="async" width="278" height="300" src="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-6-278x300.png" alt="" class="wp-image-69117" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/08/image-6-278x300.png 278w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-6-139x150.png 139w, https://kiwicloud.ninja/wp-content/uploads/2023/08/image-6.png 575w" sizes="(max-width: 278px) 100vw, 278px" /></a></figure>
</div>

If everything&#8217;s been configured correctly you should now be able to select the appropriate client SSL certificate in the virtual server **Client SSL** section and successfully be able to connect to a load balanced VCD environment.

As always, comments and feedback welcome, hope this is helpful to someone else.

Jon.