---
title: vCloud Availability 3.0 – Introduction
author: Jon Waite
type: post
date: 2019-04-22T03:29:00+00:00
url: /2019/04/vcloud-availability-3-0-introduction/
featured_image: https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAlogo-150x85.jpg
categories:
  - vCloud Availability
  - vCloud Director
  - VMware

---
VMware has recently released version 3.0 of [vCloud Availability][1] (vCAv) ([Release Notes][2]) which allows vCloud Service Providers to offer a variety of VM protection and migration services to their tenant customers. vCAv 3.0 combines features previously available in 3 separate VMware products (vCloud Availability Cloud-to-Cloud DR, vCloud Availability for vCloud Director and vCloud Extender) and allows:

<ul class="wp-block-list">
  <li>
    Protect/replicate and failover VMs to/from on-premise vSphere environments to a vCloud Service Provider.
  </li>
  <li>
    Protect/replicate and failover VMs between 2 virtual datacenters provided by a vCloud Service Provider (these would generally be in 2 distinct geographic locations).
  </li>
  <li>
    Migrate VMs to/from on-premise vSphere environments and a vCloud Service Provider.
  </li>
</ul>

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAlogo-800x452.jpg" alt="" class="wp-image-929" width="600" height="339" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAlogo-800x452.jpg 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAlogo-300x169.jpg 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAlogo-768x434.jpg 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAlogo-150x85.jpg 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAlogo-250x141.jpg 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAlogo.jpg 965w" sizes="(max-width: 600px) 100vw, 600px" /><figcaption>vCloud Availability 3.0 Functions (Image is (c)VMware 2019)</figcaption></figure>
</div>

vCloud Availability 3.0 (vCAv) also supports advanced functionality usually reserved for products such as VMware Site Recovery Manager (SRM) such as allowing VM network information to be changed during failover to ensure VMs can connect to the destination network when failed-over or migrated. The tenant administrative portal is tightly integrated into VMware vCloud Director allowing full control of VM replication tasks in the same interface used by tenants to administer their virtual machines.

Service Providers can define policies and apply these on a per-tenant basis to control items such as:

<ul class="wp-block-list">
  <li>
    How many customer VMs can be replicated (a fixed number of VMs or &#8216;unlimited&#8217;).
  </li>
  <li>
    What the minimum configurable RPO interval is for VM replication (as low as 5 minutes for vSphere 6.5+ environments and up to 24 hours).
  </li>
  <li>
    How many snapshots of each VM can be retained (from 1 to 24).
  </li>
</ul>

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/vcav-policies.png" alt="" class="wp-image-1019" width="435" height="386" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/vcav-policies.png 580w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vcav-policies-300x266.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vcav-policies-150x133.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vcav-policies-169x150.png 169w" sizes="(max-width: 435px) 100vw, 435px" /><figcaption>vCloud Availability Policy Definition</figcaption></figure>
</div>

Since the release of vCAv 3.0 I&#8217;ve been deploying and testing the solution components, this is the first part in a series of posts is designed to emulate a complete &#8216;real-world&#8217; deployment consisting of 2 distinct cloud provider sites and a &#8216;customer&#8217; on-premises infrastructure so I can detail all of the deployment, configuration and end-user usage scenarios across these.

To configure a production-realistic environment, I have deployed separate vCAv appliances for the &#8216;cloud&#8217;, &#8216;replicator&#8217; and &#8216;tunnel&#8217; functions, a typical service provider network diagram with the ports used by vCAv for communication is shown in the diagram below. Note that in an actual production implementation the &#8216;tunnel&#8217; appliance would generally be deployed into a DMZ network with the &#8216;cloud&#8217; (Replication Manager and vApp Replication Manager) and &#8216;replicator&#8217; appliances deployed into the Service Provider management network.

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/vCA-High-Level-Architecture-2-2.svg" alt="" class="wp-image-1027" /><figcaption>vCloud Availability 3.0 Network Architecture & Ports</figcaption></figure>
</div>

This concludes the first post in this series, in future posts I aim to cover:

<ul class="wp-block-list">
  <li>
    Deployment and configuration of vCAv appliances into a Cloud Service Provider
  </li>
  <li>
    Pairing Cloud Provider Sites, Defining VM replication policies and assigning these to tenants
  </li>
  <li>
    On-premise deployment and configuration into a customer vSphere cluster
  </li>
  <li>
    Protecting / replicating VMs from Cloud to Cloud, On-Premise to Cloud and Cloud to On-Premise (migration, failover and failback)
  </li>
  <li>
    Monitoring and Troubleshooting vCloud Availability services
  </li>
  <li>
    Conclusions, References and further reading
  </li>
</ul>

As always, corrections, comments and feedback are always appreciated.

Jon.

 [1]: https://docs.vmware.com/en/VMware-vCloud-Availability/
 [2]: https://docs.vmware.com/en/VMware-vCloud-Availability/3.0/rn/VMware-vCloud-Availability-30-Release-Notes.html