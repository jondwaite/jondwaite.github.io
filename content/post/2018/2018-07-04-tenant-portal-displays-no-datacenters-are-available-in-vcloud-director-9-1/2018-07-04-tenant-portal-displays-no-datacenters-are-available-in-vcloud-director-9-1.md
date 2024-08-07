---
title: Tenant Portal Displays ‘No Datacenters are available’ in vCloud Director 9.1
author: Jon Waite
type: post
date: 2018-07-03T22:35:45+00:00
url: /2018/07/tenant-portal-displays-no-datacenters-are-available-in-vcloud-director-9-1/
categories:
  - vCloud Director
  - VMware
tags:
  - Configuration
  - DNS
  - Tenant Portal
  - Troubleshooting

---
We had an issue recently when updating our vCloud Director environment to v9.1 where the new tenant portal would show ‘No Datacenters are available’ for every tenant even though the remainder of the site worked correctly (and other tabbed options like the Service Library & catalogs worked fine). Initially we suspected that our SSL certificate chain or public URI’s were set incorrectly.

Adrian Begg has a great blog post here: <a title="http://www.pigeonnuggets.com/2018/03/vcloud-director-9-1-tenant-portal-displays-no-datacenters-available-after-upgrade/" href="http://www.pigeonnuggets.com/2018/03/vcloud-director-9-1-tenant-portal-displays-no-datacenters-available-after-upgrade/" target="_blank" rel="noopener" class="broken_link">http://www.pigeonnuggets.com/2018/03/vcloud-director-9-1-tenant-portal-displays-no-datacenters-available-after-upgrade/</a> which details this issue and how to ensure the correct settings are applied, however in our case this didn’t resolve our issue.

Eventually an offhand remark in a slack channel by <a href="https://fojta.wordpress.com/" target="_blank" rel="noopener">Tom Fojta</a> put me on the right track to solving the issue, I’ve written this post up in case anyone else comes across the same issue. If you’re impatient and want to know the solution – it’s DNS (isn’t it always DNS?), but that’s jumping ahead a bit.

In our environment we have 3 vCloud Director cell servers behind a load balancer, we also load-balance internally so that our management environment can talk to the vCD API and we can conduct testing of the environment without necessarily having it open to the public internet. The arrangement looks logically like this:

&nbsp;

[<img loading="lazy" decoding="async" style="margin: 5px auto; float: none; display: block;" title="vCD Load Balancing" src="https://kiwicloud.ninja/wp-content/uploads/2018/07/vcd-load-balancing_thumb.png" alt="vCloud Director Load Balancing" width="800" height="400" />][1]

Users from the internet accessing ‘portal.cloud.com’ get redirected to one of the vCD cell servers (and if one of them is unavailable the monitoring on the Load Balancer doesn’t direct requests there). The same happens for internal users, but in this case the ‘portal.cloud.com’ DNS entry has been overridden to point at the internal (192.168.0.10) address to allow connectivity to the cells even if the external LB or internet link is unavailable.

The issue in our environment was that the cell servers themselves use DNS to access the vCloud API – and they use the public URL specified in the vCloud Director configuration.

The cell servers were configured with our internal DNS servers, so when they attempted to access the public URL (‘portal.cloud.com’) were being given the internal Load Balancer address (192.168.0.10). For reasons we’re still exploring, this didn’t allow them to get a response from the vCD API and resulted in the ‘No Datacenters are available’ error in the tenant portal.

The fix turned out to be reasonably simple – on each cell server we added an entry to the /etc/hosts file to resolve the public URL to the cell’s own IP address, so on cell 01:

192.168.0.11    portal.cloud.com

On cell02:

192.168.0.12   portal.cloud.com

And on cell03:

192.168.0.13    portal.cloud.com

Once we’d made this change the tenant portals began functioning correctly (note that no restart of the cell servers or vCloud Director services was required).

What I assume is happening is that when the internal load balancer responds the the request it gives out a different cell server’s address (since the ‘source’ of the request will be a cell server) and that cell server has no knowledge of the session being used by the original cell and so responds incorrectly (either with nothing, or with an error). Not sure if this is actually a bug, or just something to be aware of, but either way overriding name resolution in this way fixes the issue. Note that simply using ‘localhost’ or 127.0.0.1 for the hosts file entry doesn’t work since the vCloud web server doesn’t respond on the loopback interface in the default configuration.

Just posting this here in the hope it will save someone else any frustration caused by this issue.

Jon.

 [1]: https://kiwicloud.ninja/wp-content/uploads/2018/07/vcd-load-balancing.png