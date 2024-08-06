---
title: Quick Post – Network Error Deploying VMware Cloud Director Availability 4.0 Virtual Appliances
author: Jon Waite
type: post
date: 2020-06-04T02:29:20+00:00
url: /2020/06/quick-post-network-error-deploying-vmware-cloud-director-availability-virtual-appliances/
categories:
  - vCloud Availability

---
Just a quick post covering an issue I discovered when deploying new appliances for the recently released VMware Cloud Director Availability (VCDA) solution (the product formerly known as vCloud Availability).

When I deployed the first management appliance I was a bit puzzled to see this on the console during first power-up (`Failed to start vCAv network initializer`):

 

<div class="wp-block-image">
  <figure class="aligncenter size-full is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2020/06/03-VCDA-deploy-console-01.png" alt="" class="wp-image-1188" width="602" height="451" srcset="https://kiwicloud.ninja/wp-content/uploads/2020/06/03-VCDA-deploy-console-01.png 802w, https://kiwicloud.ninja/wp-content/uploads/2020/06/03-VCDA-deploy-console-01-300x225.png 300w, https://kiwicloud.ninja/wp-content/uploads/2020/06/03-VCDA-deploy-console-01-800x600.png 800w, https://kiwicloud.ninja/wp-content/uploads/2020/06/03-VCDA-deploy-console-01-768x576.png 768w, https://kiwicloud.ninja/wp-content/uploads/2020/06/03-VCDA-deploy-console-01-150x112.png 150w, https://kiwicloud.ninja/wp-content/uploads/2020/06/03-VCDA-deploy-console-01-200x150.png 200w" sizes="(max-width: 602px) 100vw, 602px" /></figure>
</div>

And then when the startup had completed, the console was showing no IP address had been assigned to the appliance:

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2020/06/04-VCDA-deploy-console-02-800x600.png" alt="" class="wp-image-1189" width="600" height="450" srcset="https://kiwicloud.ninja/wp-content/uploads/2020/06/04-VCDA-deploy-console-02-800x600.png 800w, https://kiwicloud.ninja/wp-content/uploads/2020/06/04-VCDA-deploy-console-02-300x225.png 300w, https://kiwicloud.ninja/wp-content/uploads/2020/06/04-VCDA-deploy-console-02-768x576.png 768w, https://kiwicloud.ninja/wp-content/uploads/2020/06/04-VCDA-deploy-console-02-150x112.png 150w, https://kiwicloud.ninja/wp-content/uploads/2020/06/04-VCDA-deploy-console-02-200x150.png 200w, https://kiwicloud.ninja/wp-content/uploads/2020/06/04-VCDA-deploy-console-02.png 802w" sizes="(max-width: 600px) 100vw, 600px" /></figure>
</div>

Which was odd, as I was sure I&#8217;d specified the correct IP information during appliance deployment. Logging in to the appliance and looking at the status of the `h4network.service` showed errors:

 

<div class="wp-block-image">
  <figure class="aligncenter size-full is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2020/06/05-VCDA-deploy-console-03.png" alt="" class="wp-image-1190" width="600" height="229" srcset="https://kiwicloud.ninja/wp-content/uploads/2020/06/05-VCDA-deploy-console-03.png 800w, https://kiwicloud.ninja/wp-content/uploads/2020/06/05-VCDA-deploy-console-03-300x114.png 300w, https://kiwicloud.ninja/wp-content/uploads/2020/06/05-VCDA-deploy-console-03-768x293.png 768w, https://kiwicloud.ninja/wp-content/uploads/2020/06/05-VCDA-deploy-console-03-150x57.png 150w, https://kiwicloud.ninja/wp-content/uploads/2020/06/05-VCDA-deploy-console-03-250x95.png 250w" sizes="(max-width: 600px) 100vw, 600px" /></figure>
</div>

The errors appeared to indicate an issue with the MTU network property from the appliance (&#8216;`sysboot.extract_ovf_property(Properties, 'net.mtu'`) in the `sysboot.py` script. So I did some more digging and tried deploying again with different MTU values, none of which got me any further. I then took a step-back and looked again at the network properties I&#8217;d entered for the appliance:

 

<div class="wp-block-image">
  <figure class="aligncenter size-full is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2020/06/01-VCDA-deploy-IP-information.png" alt="" class="wp-image-1191" width="656" height="542" srcset="https://kiwicloud.ninja/wp-content/uploads/2020/06/01-VCDA-deploy-IP-information.png 874w, https://kiwicloud.ninja/wp-content/uploads/2020/06/01-VCDA-deploy-IP-information-300x248.png 300w, https://kiwicloud.ninja/wp-content/uploads/2020/06/01-VCDA-deploy-IP-information-800x661.png 800w, https://kiwicloud.ninja/wp-content/uploads/2020/06/01-VCDA-deploy-IP-information-768x634.png 768w, https://kiwicloud.ninja/wp-content/uploads/2020/06/01-VCDA-deploy-IP-information-150x124.png 150w, https://kiwicloud.ninja/wp-content/uploads/2020/06/01-VCDA-deploy-IP-information-182x150.png 182w" sizes="(max-width: 656px) 100vw, 656px" /></figure>
</div>

Hands up if you can spot the error before scrolling down for the answer&#8230;

In my rush to deploy a shiny new VCDA instance, I hadn&#8217;t properly read the information correctly against the &#8216;Address&#8217; entry:

<div class="wp-block-image">
  <figure class="aligncenter size-full is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2020/06/02-VCDA-deploy-IP-information.png" alt="" class="wp-image-1192" width="656" height="542" srcset="https://kiwicloud.ninja/wp-content/uploads/2020/06/02-VCDA-deploy-IP-information.png 874w, https://kiwicloud.ninja/wp-content/uploads/2020/06/02-VCDA-deploy-IP-information-300x248.png 300w, https://kiwicloud.ninja/wp-content/uploads/2020/06/02-VCDA-deploy-IP-information-800x661.png 800w, https://kiwicloud.ninja/wp-content/uploads/2020/06/02-VCDA-deploy-IP-information-768x634.png 768w, https://kiwicloud.ninja/wp-content/uploads/2020/06/02-VCDA-deploy-IP-information-150x124.png 150w, https://kiwicloud.ninja/wp-content/uploads/2020/06/02-VCDA-deploy-IP-information-182x150.png 182w" sizes="(max-width: 656px) 100vw, 656px" /></figure>
</div>

I&#8217;d provided a valid IP address for the appliance, but not what was being asked for &#8211; a CIDR address including the subnet mask. After changing the value shown to one appropriate for my network (192.168.0.20/24 in this case) I was able to re-deploy the appliance and the networking came up fine as expected.

Hopefully this will teach me to be a bit more careful in future reading the descriptions for vApp properties when deploying appliances and hopefully will be helpful to anyone else facing the same issue.

Jon.