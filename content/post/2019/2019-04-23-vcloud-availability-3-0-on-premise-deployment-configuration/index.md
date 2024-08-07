---
title: 'vCloud Availability 3.0 – On-Premise Deployment & Configuration'
author: Jon Waite
type: post
date: 2019-04-22T23:21:03+00:00
url: /2019/04/vcloud-availability-3-0-on-premise-deployment-configuration/
categories:
  - vCloud Availability
  - vCloud Director
  - VMware

---
In the first 3 parts of this series I detailed the configuration and deployment of vCAv 3.0 into a service provider site. In this 4th part I show the deployment and configuration of vCAv into a tenant on-premise infrastructure. This allows appropriately configured tenants to protect on-premise VMs to a cloud provider as well as protect cloud-hosted VMs back to their own on-premise infrastructure.

In this configuration I will use the already-configured Christchurch lab cloud environment as the endpoint and configure vCAv into an on-premise infrastructure at my test 'Tyrell' tenant environment which is configured with its own small vSphere SDDC based on vCenter, ESXi and VSAN storage. Note that NSX networking is not required in the tenant site, and only outbound tcp/443 (https) connectivity is required between the tenant and cloud provider site which makes vCAv almost trivially easy for customers to deploy into their datacenters.

The VMware documentation page for deploying vCAv at a tenant/on-premise site is available [here][1], the process below show the configuration of the vCAv tenant appliance and configuring the appliance once deployed to connect to a cloud provider site.

Download the OnPrem version of the vCAv appliance from my.vmware.com, note that this is different from the image used to deploy a Cloud Provider site and is listed under the 'Drivers & Tools' tab in my.vmware.com:

<div class="wp-block-image">
  <figure class="aligncenter"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/04-onprem-appliance-download.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" width="800" height="109" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/04-onprem-appliance-download-800x109.png" alt="" class="wp-image-1055" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/04-onprem-appliance-download-800x109.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/04-onprem-appliance-download-300x41.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/04-onprem-appliance-download-768x104.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/04-onprem-appliance-download-150x20.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/04-onprem-appliance-download-250x34.png 250w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>

Login to the on-premise vCenter and select the option to deploy an OVF Template, select the downloaded OnPrem appliance:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-01-e1555971052626.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-01-e1555971052626-800x323.png" alt="" class="wp-image-1054" width="600" height="242" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-01-e1555971052626-800x323.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-01-e1555971052626-300x121.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-01-e1555971052626-768x310.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-01-e1555971052626-150x61.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-01-e1555971052626-250x101.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-01-e1555971052626.png 876w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Specify the VM name to be assigned to the vCAv OnPrem appliance and select the datacenter where it will be deployed:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-02-e1555971544770.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-02-e1555971544770-800x269.png" alt="" class="wp-image-1056" width="600" height="202" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-02-e1555971544770-800x269.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-02-e1555971544770-300x101.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-02-e1555971544770-768x259.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-02-e1555971544770-150x51.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-02-e1555971544770-250x84.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-02-e1555971544770.png 876w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Next select the vSphere cluster where the vCAv appliance will run:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-03-e1555971636807.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-03-e1555971636807-800x217.png" alt="" class="wp-image-1057" width="600" height="163" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-03-e1555971636807-800x217.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-03-e1555971636807-300x82.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-03-e1555971636807-768x209.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-03-e1555971636807-150x41.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-03-e1555971636807-250x68.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-03-e1555971636807.png 876w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

The next screen allows you to review the template details prior to deployment:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-04.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-04-800x661.png" alt="" class="wp-image-1058" width="600" height="496" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-04-800x661.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-04-300x248.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-04-768x635.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-04-150x124.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-04-181x150.png 181w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-04.png 876w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

You must agree to the VMware EULA agreement before proceeding:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-05.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-05-800x661.png" alt="" class="wp-image-1059" width="600" height="496" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-05-800x661.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-05-300x248.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-05-768x635.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-05-150x124.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-05-181x150.png 181w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-05.png 876w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Chose the datastore and storage policy for the vCAv OnPrem appliance deployment (in this example I only have a single VSAN datastore to select from):

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-06.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-06-800x661.png" alt="" class="wp-image-1060" width="600" height="496" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-06-800x661.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-06-300x248.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-06-768x635.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-06-150x124.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-06-181x150.png 181w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-06.png 876w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Select the network for the vCAv appliance to connect on and the IP assignment type:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-07.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-07-800x661.png" alt="" class="wp-image-1061" width="600" height="496" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-07-800x661.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-07-300x248.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-07-768x635.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-07-150x124.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-07-181x150.png 181w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-07.png 876w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Assign the initial 'root' user password (as with the Cloud appliance, this will be forced to change on first login to the appliance UI) and configure appropriate networking settings - IP address, subnet mask, default gateway, DNS servers, DNS domain and NTP server:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-08.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-08-800x661.png" alt="" class="wp-image-1063" width="600" height="496" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-08-800x661.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-08-300x248.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-08-768x635.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-08-150x124.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-08-181x150.png 181w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-08.png 876w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Review the summary screen and click 'Finish' to initiate the appliance deployment:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-09.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-09-800x661.png" alt="" class="wp-image-1064" width="600" height="496" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-09-800x661.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-09-300x248.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-09-768x635.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-09-150x124.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-09-181x150.png 181w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-OVF-deploy-09.png 876w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Once the appliance deployment has completed, you can power it on and access the configuration UI at https://<appliance-deployed-ip> to continue with configuration. Once you have logged in and changed the 'root' user password you will be shown the screen shown below:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-01.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-01-800x357.png" alt="" class="wp-image-1066" width="600" height="268" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-01-800x357.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-01-300x134.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-01-768x343.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-01-150x67.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-01-250x112.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-01.png 896w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Click to run the initial setup wizard and enter a name for the on-premise site:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-02.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-02-800x251.png" alt="" class="wp-image-1067" width="600" height="188" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-02-800x250.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-02-300x94.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-02-768x241.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-02-150x47.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-02-250x79.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-02.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Enter the Lookup service address and authentication details for the **local** (on-prem) vCenter/PSC infrastructure (in this example I'm once again using vCenter with an embedded PSC):

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-03.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-03-800x251.png" alt="" class="wp-image-1068" width="600" height="188" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-03-800x250.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-03-300x94.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-03-768x241.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-03-150x47.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-03-250x79.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-03.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Next specify the URI for the public API endpoint for the cloud-provider instance of vCloud Availability (**not** the vCloud Director public endpoint) and provide login credentials to the tenant organization within that cloud environment, you will need to confirm the provided SSL certificate as the connection is made.

Note: If you select the 'Allow Access from Cloud' option, then administrators in the Cloud Provider will gain capability in the local vCenter/vSphere environment - here's what the VMware documentation says on this:

_&#8220;By selecting this option you allow the cloud provider and the organization administrators to execute the following operations from the vCloud Availability Portal without authenticating to the on-premises site._

<ul class="wp-block-list">
  <li>
    <em>Discover on-premises workloads and replicate them to the cloud.</em>
  </li>
  <li>
    <em>Reverse existing replications to the on-premises site.</em>
  </li>
  <li>
    <em>Replicate cloud workloads to the on-premises site.</em>
  </li>
</ul>

_By leaving this option deselected, only users authenticated to the on-premises vCloud Availability Portal can configure new replications and existing replications cannot be reversed from the vCloud Availability Portal.&#8221;_

Make sure you understand the implications of selecting this option (or not selecting this option) when configuring the appliance and set it appropriately. In my lab environment I enabled the setting.

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-04.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-04-800x251.png" alt="" class="wp-image-1069" width="600" height="188" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-04-800x250.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-04-300x94.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-04-768x241.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-04-150x47.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-04-250x79.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-04.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Confirm / deny participation in VMware CEIP in the next screen:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-05.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-05-800x284.png" alt="" class="wp-image-1070" width="600" height="213" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-05-800x284.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-05-300x106.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-05-768x272.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-05-150x53.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-05-250x89.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-05.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

In the confirmation screen, you can use the slider to continue to the 'Configure local placement' dialogs on completion of the initial OnPrem appliance configuration. If you chose you can complete this process separately later, but in this example I chose to continue and configure local VM placement immediately:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-06b.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-06b-800x346.png" alt="" class="wp-image-1074" width="600" height="260" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-06b-800x346.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-06b-300x130.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-06b-768x332.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-06b-150x65.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-06b-250x108.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Config-06b.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

The placement configuration allows you to select the environment that will be used by VMs replicated from the cloud to the OnPrem environment, in the first screen select the VM folder destination where the VMs will appear:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-01.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-01-800x224.png" alt="" class="wp-image-1076" width="600" height="168" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-01-800x224.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-01-300x84.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-01-768x215.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-01-150x42.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-01-250x70.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-01.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Next select the compute cluster where the VMs will be registered:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-02.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-02-800x224.png" alt="" class="wp-image-1077" width="600" height="168" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-02-800x224.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-02-300x84.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-02-768x215.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-02-150x42.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-02-250x70.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-02.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Next select the default network to which the replicated VMs will be attached:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-03.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-03-800x224.png" alt="" class="wp-image-1078" width="600" height="168" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-03-800x224.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-03-300x84.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-03-768x215.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-03-150x42.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-03-250x70.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-03.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Select the vSphere Datastore where the replicated VM disks will be stored:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-04.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-04-800x224.png" alt="" class="wp-image-1079" width="600" height="168" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-04-800x224.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-04-300x84.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-04-768x215.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-04-150x42.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-04-250x70.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-04.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

Finally review the supplied details and complete the placement configuration:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-05.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-05-800x224.png" alt="" class="wp-image-1080" width="600" height="168" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-05-800x224.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-05-300x84.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-05-768x215.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-05-150x42.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-05-250x70.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Placement-05.png 1156w" sizes="(max-width: 600px) 100vw, 600px" /></a></figure>
</div>

The 'Configuration' tab in the appliance should now show the configured values as shown below:

<div class="wp-block-image">
  <figure class="aligncenter"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-01.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" width="800" height="453" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-01-800x453.png" alt="" class="wp-image-1081" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-01-800x453.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-01-300x170.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-01-768x435.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-01-150x85.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-01-250x141.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-01.png 1937w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>

Clicking the 'System Monitoring' tab should show connectivity to all services and to the remove Service Provider cloud:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-02.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-02-800x453.png" alt="" class="wp-image-1082" width="800" height="453" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-02-800x453.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-02-300x170.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-02-768x435.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-02-150x85.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-02-250x141.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-02.png 1937w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>

Signing into the local vCenter environment should now display the banner (shown below) as the vCAv plugin is registered into the local vCenter:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-03.png" alt="" class="wp-image-1083" width="1028" height="74" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-03.png 1370w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-03-300x22.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-03-768x55.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-03-800x58.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-03-150x11.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-03-250x18.png 250w" sizes="(max-width: 1028px) 100vw, 1028px" /></figure>
</div>

Clicking the 'Refresh Browser' button and going to the 'Home' link in vCenter will now show a new menu entry for vCloud Availability:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-04.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-04-800x588.png" alt="" class="wp-image-1084" width="600" height="441" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-04-800x588.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-04-300x221.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-04-768x565.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-04-150x110.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-04-204x150.png 204w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-04.png 805w" sizes="(max-width: 600px) 100vw, 600px" /></a><figcaption>vCloud Availability tab in vCenter HTML5 UI post-installation & Configuration of the OnPrem vCAv Appliance</figcaption></figure>
</div>

Selecting the 'vCloud Availability' link in vCenter will open a new panel showing the vCloud Availability interface:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><a href="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-05.png" target="_blank" rel="noreferrer noopener"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-05-800x308.png" alt="" class="wp-image-1085" width="800" height="308" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-05-800x308.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-05-300x115.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-05-768x295.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-05-150x58.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-05-250x96.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/05-Post-Config-05.png 1937w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>

That completes the configuration of an on-premise connection to vCloud Availability - at this point we can now replicate VMs both to and from the Cloud Service Provider infrastructure to our own vSphere cluster. Although there are quite a few steps in the process, I hope you can see that the configuration of the OnPrem appliance is actually very straightforward and easy. A particular advantage compared to previous deployments with other products such as vCloud Availability and vCloud Extender is that no inbound firewall or NAT rules are required in the vCAv OnPrem configuration with v3.0.

This concludes the 4th part of this series looking at vCloud Availability 3.0, in the next part of the series now that we have both Cloud-to-Cloud (Parts 2 & 3) and OnPrem-to-Cloud (this part) configurations completed I'll look at configuring VM replication protection and failover/migration of replicated VMs.

As always, corrections, comments and feedback welcome!

Jon

 [1]: https://docs.vmware.com/en/VMware-vCloud-Availability/3.0/com.vmware.vcav.onprem.install.config.doc/GUID-E526EEAF-8CB5-4D8C-A84F-CA2B6E3D95C6.html