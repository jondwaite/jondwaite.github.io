---
title: VMware Container Solutions
author: Jon Waite
type: post
date: 2018-02-24T04:24:00+00:00
url: /2018/02/vmware-container-solutions/
categories:
  - Containers
  - vCloud Director
  - VMware
tags:
  - Containerisation
  - CSE
  - Docker
  - Kubernetes
  - Microservices
  - PKS
  - VIC

---
VMware appears to have gone a little ‘mad’ with regards to containerisation (or containerization for any American readers) lately. Last week saw the release of Pivotal Container Service (PKS) as launched at VMworld 2017 US back in August. With this there are now a total of three VMware technologies all enabling customers to run micro-service applications in their environments. So why three different products to do the same thing? Well, they are targeted at different environments and use-cases, and actually it makes a lot of sense for VMware to have solutions for all 3 scenarios. Of course there’s always the 4th option of building your own container hosting platform from scratch on a VMware platform, but lets concentrate for now on those provided by VMware.

So what are the available solutions?

#### Pivotal Container Service (PKS)

This was announced at VMworld 2017 and recently became available for download. PKS a full stack solution to manage both initial formation of clusters to support containerised applications and manage their ‘day 2’ operations once deployed. While PKS could be deployed in an Enterprise environment (and may be for organisations using containerised applications at significant scale) it appears to be more targetted towards cloud service providers wishing to offer a managed/hosted platform for multiple tenants.

#### vSphere Integrated Containers (VIC)

VIC has been around for a while now (this was based on VMware’s Project Bonneville which started back in 2015), recently VIC has been updated to v1.3.1 and gained the capability to use Docker hosts natively at version 1.2 (prior to this VMware Host Containers had to be used). VIC supports vSphere version 6.0 and upwards and is primarily targeted at Enterprise customers wishing to provide a managed container hosting environment within their own infrastructure.

#### Container Service Extension (CSE) for vCloud Director

Sitting somewhat in between the other offerings, VMware has also released CSE via an open source Github repository. CSE is targeted at Service Providers using VMware’s vCloud Director platform who wish to make delivering container hosting to tenants much easier. It provides an extension to vCloud Director which allows the creation and maintenance of clusters of VMs providing Docker in Kubernetes clusters.

### Comparing the solutions

The table below shows a summary of the options

&nbsp;

<table border="1" width="100%" cellspacing="0" cellpadding="2">
  <tr>
    <td valign="top" width="25%">
      <strong>Solution</strong>
    </td>
    
    <td valign="top" width="25%">
      <strong>Pivotal Container Service (PKS)</strong>
    </td>
    
    <td valign="top" width="25%">
      <strong>vSphere Integrated Containers (VIC)</strong>
    </td>
    
    <td valign="top" width="25%">
      <strong>Container Service Extension (CSE)</strong>
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Current release / link
    </td>
    
    <td valign="top" width="25%">
      <a href="https://network.pivotal.io/products/pivotal-container-service">v1.0.0 GA</a>
    </td>
    
    <td valign="top" width="25%">
      <a href="https://www.vmware.com/products/vsphere/integrated-containers.html" class="broken_link">v1.3.1</a>
    </td>
    
    <td valign="top" width="25%">
      <a href="https://vmware.github.io/container-service-extension/">v0.4.2</a>
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Container Runtime
    </td>
    
    <td valign="top" width="25%">
      Docker
    </td>
    
    <td valign="top" width="25%">
      Docker & Virtual Container Host (VCH)
    </td>
    
    <td valign="top" width="25%">
      Docker<sup>[1]</sup>
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Container Management
    </td>
    
    <td valign="top" width="25%">
      Kubernetes
    </td>
    
    <td valign="top" width="25%">
      VMware Admiral
    </td>
    
    <td valign="top" width="25%">
      Kubernetes<sup>[1]</sup>
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Container OS
    </td>
    
    <td valign="top" width="25%">
      BOSH
    </td>
    
    <td valign="top" width="25%">
      Virtual Container Host (Photon OS based)
    </td>
    
    <td valign="top" width="25%">
      Any (Ubuntu & Photon provided)
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Container Registry
    </td>
    
    <td valign="top" width="25%">
      VMware Harbor
    </td>
    
    <td valign="top" width="25%">
      VMware Harbor
    </td>
    
    <td valign="top" width="25%">
      Any (None provided)
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Deployed to
    </td>
    
    <td valign="top" width="25%">
      Bare metal / VM
    </td>
    
    <td valign="top" width="25%">
      Bare metal / vSphere VM
    </td>
    
    <td valign="top" width="25%">
      vCloud Director Virtual Datacenter (VDC)
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Multi-tenant Supported
    </td>
    
    <td valign="top" width="25%">
      Yes
    </td>
    
    <td valign="top" width="25%">
      Yes
    </td>
    
    <td valign="top" width="25%">
      Yes
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Network Support
    </td>
    
    <td valign="top" width="25%">
      VMware NSX-T
    </td>
    
    <td valign="top" width="25%">
      vSphere & VMware NSX-V
    </td>
    
    <td valign="top" width="25%">
      Org VDC Networks (vCloud Director) / VMware NSX-V
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Licensing / Support
    </td>
    
    <td valign="top" width="25%">
      Open Source, Paid Support available from Pivotal
    </td>
    
    <td valign="top" width="25%">
      Open Source, vSphere S&S Support covers VIC
    </td>
    
    <td valign="top" width="25%">
      Open Source, Service Provider Support
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="25%">
      Primarily Targeted At
    </td>
    
    <td valign="top" width="25%">
      Service Providers & Enterprise using containers at scale
    </td>
    
    <td valign="top" width="25%">
      Enterprise
    </td>
    
    <td valign="top" width="25%">
      Service Provider / vCloud Tenants
    </td>
  </tr>
</table>

<sup>[1]</sup> CSE allows service providers to provide any versions of Docker and Kubernetes in their templates. This can allow much more up-to-date versions than those supported in PKS or VIC.

### CSE deployment for a Service Provider

I’ve recently been involved with deploying CSE to our own vCloud Director hosting platform, the VMware <a href="https://vmware.github.io/container-service-extension/" target="_blank" rel="noopener">github.io</a> page is extremely useful and well documented to help get up and running with CSE so I won’t repeat this here.

The main advantages it offered us as a service provider:

**No new billing / Integration required**  
This is a huge deal for most service providers, it can be time-consuming (and therefore expensive) to integrate any new platform offering, not just in the time taken to deploy the components and get them all working correctly (including alerting, monitoring etc.) but what is often overlooked is the additional effort required to correctly meter platform consumption and ensure that customer bills are correctly prepared and reflect the resources their environments have consumed. Taking the ‘full stack’ of PKS and offering this as a service would involve considerable work, but with CSE this workload is effectively neutralised since the clusters deployed are directly into tenant virtual datacenters (VDCs) and service providers will already be metering and billing customers for resources consumed in tenant VDCs.

**No new licensing**  
As there is no additional licensing for CSE this makes it extremely easy to deploy in a service provider platform.

**No new security model**  
Since all tenant interaction with CSE is via the vCloud Director API, there is very little work required (if any) to publish the service to customers since most Service Providers will already be making the vCD API accessible to their tenants. Additionally, since the CSE service itself integrates directly into vCloud Director’s RabbitMQ backend it is likely that very few security or firewall changes are required either.

**Flexible environment**  
One of the really nice aspects of CSE is that the templates made available to tenants to deploy into clusters are fully customisable. This means that service providers can chose to offer additional templates beyond the 2 examples provided ‘out of the box’ with CSE. For example, if a Service Provider wishes to offer a ‘bleeding edge’ template which has the absolute latest releases of Docker and Kubernetes (and maybe add additional packages to the deployed images to include Harbor and maybe a ceph or glusterfs client) this is reasonably straightforward and easy to do. The downside of this is that maintenance and updating these templates has to be performed regularly to ensure that they include all appropriate bug-fixes and security patches and updates.

Note that at this time the VMware documentation doesn’t yet include instructions for modifying or adding additional CSE templates, I’ll write up a separate post on how I did this in our environment which may prove useful for others deploying CSE into their own environments.

### Other CSE Considerations

Of course no platform is ever perfect, and the following should be noted too as potential pitfalls or things to be aware of when considering CSE:

**No registry service by default**  
Both PKS and VIC provide container registry services (to deal with storing, securing, scanning and replicating container images) based on VMware’s Project Harbor which is a very nice registry system. While Harbor can be added to clusters deployed with CSE, it isn’t there by default in the templates currently provided with CSE.

**No persistent or dynamic Kubernetes volumes**  
Containers by design are meant to be ephemeral and stateless, so they shouldn’t be storing any persistent data or require backup protection. Of course most business applications (including those provided by containerised images) generally need some form of permanent/persistent storage behind them. In Kubernetes environments this is generally accomplished by the concept of persistent volumes which are mapped into containers at runtime and allow data to be retained. In CSE currently there is no provider for persistent volumes which means that external storage is required. This can however be delivered from a variety of sources – other databases running in the environment, file or object storage services etc. I’m currently looking into easy ways to add dynamically provisioned persistent volumes to a CSE cluster and will write this up as a separate post when done.

**Template maintenance**  
As mentioned previously, the templates deployed by CSE are completely flexible and can be easily customised by editing their deployment scripts, the process of maintaining the templates is reasonably manual though and requires stopping the CSE service, patching and updating the templates in a vCloud Director shared catalog and then re-enabling the CSE service. It would be nice to have a way to automate the rebuild of templates and to allow the CSE services to remain online while this is happening.

**Relative immaturity**  
The CSE service is a very ‘early’ release and a number of bugs are still being fixed. There’s nothing too serious that I’ve encountered yet, but occasionally templates will fail to build correctly (generally due to failures in 3rd party repositories) and it can take time to identify and resolve these issues. Fortunately the VMware developers have been extremely fast and active in responding to issues raised in the CSE github repository and every issue I’ve found has been very quickly fixed.

### Summary

Hopefully this post has given you an idea of the capabilities and features available in the 3 current VMware container hosting solutions and given you a better idea of what the Cloud Service Extension for vCloud Director does. I’m aiming to write some follow-up posts on CSE including how we have deployed it into our environment, how new templates can be created (and existing templates customised) and how to address some of the current missing features such as integrating Harbor as a registry service in future posts. Let me know in the comments if there are any areas you are particularly interested in and I’ll see what I can do. I’ve also written a session abstract proposal to present a Service Provider view of CSE at VMworld US 2018, so hoping that that will be accepted too.

### References / Links

Some of the components mentioned may not be familiar so I’ve provided links to each one below:

BOSH: <https://bosh.io/>  
Docker: <https://www.docker.com/>  
Harbor: <https://vmware.github.io/harbor/>  
Kubernetes: <https://kubernetes.io/>  
Ubuntu Linux: <https://www.ubuntu.com/>  
VMware Photon OS: <https://vmware.github.io/photon/>

As always, comments & corrections welcome, I’m reasonably new to the whole ‘containerised applications’ scene so there may well be inaccuracies in this post(!)

Jon.