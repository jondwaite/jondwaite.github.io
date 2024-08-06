---
title: vCloud Director Extender – Part 1 Overview
author: Jon Waite
type: post
date: 2017-10-22T23:29:08+00:00
url: /2017/10/vcloud-director-extender-part-1-overview/
categories:
  - vCloud Director
  - vCloud Director Extender
  - VMware
tags:
  - CX
  - Installation
  - vCloud Director
  - vCloud Director 9
  - vCloud Director Extender
  - vSphere 6

---
Last week VMware released version 1.0 of the new vCloud Director Extender (CX) ([link to documentation set][1]). This provides some extremely flexible options for customers to migrate servers to/from a vCloud service provider cloud platform, including the use of L2VPN to transparently stretch their on-premise networks to the cloud provider. Together with a &#8216;warm&#8217; cutover feature, this enables any customer with an appropriately configured vCloud tenancy and resources to safely and easily move their virtual servers to the most suitable hosting location with minimal application downtime.

As always, there are a few pre-requisites:

&#8211; The customer site must be running vSphere 6 Update 3 or later (6.5.0 and 6.5 Update 1 are also both supported).  
&#8211; If the customer wishes to use L2VPN network extension and is already running VMware NSX, this must be v6.2.8 or v6.3.2.  
&#8211; The cloud provider must be running vCloud Director v8.20 or v9.0.

Deployment of the replication environment is different for the Cloud Provider and tenant (as you would expect) and firewall rules and address translation need to be appropriately configured to permit the required traffic flows at both the provider and customer end.

This series of articles will detail the installation and configuration of vCloud Director Extender and is intended to be useful for both Cloud Providers needing to configure their own environments to support CX and for customers wishing to configure their environments to allow migration to/from a CX-enabled provider.

The environment that I will be describing and building through this series is shown in the graphic below, Tyrell Corporation is the client organisation and MyCloud is the Cloud Provider which Tyrell wish to use to host 3 of their production VMs (&#8216;Deckard&#8217;, &#8216;Rachael&#8217; and &#8216;Roy&#8217;). In this example Tyrell and MyCloud happen to use different internal IP network ranges, but that is not a requirement to use CX since NAT firewalls are in place at both organisations.

Since I built this environment using &#8216;real&#8217; public Internet addresses and VMware NSX edge gateways as the firewalls for both Tyrell and MyCloud, I have stripped the public IP addresses from the configurations shown in these articles, but it should be easy to see where these are substituted.

<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-248" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/CX-Overview.png" alt="" width="1068" height="366" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/CX-Overview.png 1068w, https://kiwicloud.ninja/wp-content/uploads/2017/10/CX-Overview-300x103.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/CX-Overview-768x263.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/CX-Overview-1024x351.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/CX-Overview-250x86.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/CX-Overview-150x51.png 150w" sizes="(max-width: 1068px) 100vw, 1068px" /> 

I&#8217;m expecting this series to consist of 6 parts eventually including this introduction:

Part 1 &#8211; This overview  
[Part 2 &#8211; Cloud Provider / Service Provider installation and configuration (MyCloud)][2]  
[Part 3 &#8211; Customer / Tenant installation and configuration (Tyrell)][3]  
[Part 4 &#8211; Customer / Tenant connecting to a Cloud Provider and Virtual Machine migration (Tyrell)][4]  
[Part 5 &#8211; Stretched networking (L2VPN) configurations][5]  
Part 6 &#8211; Troubleshooting

I&#8217;m still working on the later parts of this series so check back if I haven&#8217;t published all of them yet.

[Link to Part 2][2]

As always, corrections, comments and feedback are always appreciated.

Jon.

 [1]: https://docs.vmware.com/en/vCloud-Director-Extender/index.html
 [2]: http://152.67.105.113/2017/10/vcloud-director-extender-part-2-cloud-provider-setup/
 [3]: http://152.67.105.113/2017/10/vcloud-director-extender-part-3-tenant-setup/
 [4]: http://152.67.105.113/2017/10/vcloud-director-extender-part-4-connect-to-provider-vm-migration/
 [5]: http://152.67.105.113/2017/11/vcloud-director-extender-part-5-stretch-networking-l2vpn/