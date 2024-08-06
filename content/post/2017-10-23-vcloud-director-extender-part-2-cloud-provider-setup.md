---
title: vCloud Director Extender – Part 2 – Cloud Provider Setup
author: Jon Waite
type: post
date: 2017-10-22T23:29:23+00:00
url: /2017/10/vcloud-director-extender-part-2-cloud-provider-setup/
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
  - VMware
  - vSphere 6

---
In the [first part][1] of this series of articles I described the new vCloud Director Extender (CX) software released by VMware. In this article I will show the steps required to install and configure the software from a Cloud Provider perspective. Included in this will be the necessary network and firewall configuration required.

vCloud Director Extender is supplied as a single .ova appliance from the VMware download site (login required). The download is located in the 'Drivers & Tools' section of the vCloud Director for Service Providers v9.0 page:

<div id='gallery-1' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/01-download.png'><img loading="lazy" decoding="async" width="800" height="483" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/01-download-1024x618.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/01-download-1024x618.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-download-300x181.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-download-768x463.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-download-250x150.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-download-150x90.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-download.png 1053w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The ova file will generate the 3 different server components required to create a functional deployment:

<table>
  <tr>
    <td>
      CX Cloud Service
    </td>
    
    <td>
      The main vCloud Director Extender appliance, this is used to provide the UI for setup/configuration. This is the appliance initially deployed from the vCloud Director Extender appliance download package.
    </td>
  </tr>
  
  <tr>
    <td>
      Cloud Continuity Manager (CCM)
    </td>
    
    <td>
      This component (also known as the 'Replicator Manager') is the operational manager of the deployment. CCM only runs in provider deployments and manages the replicator (CCE) appliances. CCM appliances are deployed and managed by the CX appliance (no additional download is required).
    </td>
  </tr>
  
  <tr>
    <td>
      Cloud Continuity Engine (CCE)
    </td>
    
    <td>
      This component (also known as the 'Replicator') is the transfer engine that deals with data transfers between the customer and provider environments. CCE runs in both the provider and client environments. CCE appliances are deployed and managed by the CX appliance (no additional download is required).
    </td>
  </tr>
</table>

The downloaded CX appliance is deployed from vCenter, the first selection allows you to specify the VM name and datacenter/folder location to deploy. In most service providers this would likely be the management cluster for their environment (as opposed to resource vcenters used for customer workloads)

<div id='gallery-2' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/02-deploy-01-destination.png'><img loading="lazy" decoding="async" width="800" height="469" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/02-deploy-01-destination.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/02-deploy-01-destination.png 963w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-deploy-01-destination-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-deploy-01-destination-768x451.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-deploy-01-destination-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-deploy-01-destination-150x88.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next you select which cluster/resource pool to deploy the CX appliance into:

<div id='gallery-3' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-02-cluster.png'><img loading="lazy" decoding="async" width="800" height="469" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-02-cluster.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-02-cluster.png 961w, https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-02-cluster-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-02-cluster-768x450.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-02-cluster-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-02-cluster-150x88.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

A Review screen is presented which allows you to confirm the ova details:

<div id='gallery-4' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-03-review.png'><img loading="lazy" decoding="async" width="800" height="469" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-03-review.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-03-review.png 963w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-03-review-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-03-review-768x451.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-03-review-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-03-review-150x88.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

And of course we have to read/accept the license agreement:

<div id='gallery-5' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-04-license.png'><img loading="lazy" decoding="async" width="800" height="469" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-04-license.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-04-license.png 963w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-04-license-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-04-license-768x450.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-04-license-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-04-license-150x88.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next we select the datastore location for deployment:

<div id='gallery-6' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-05-storage-location.png'><img loading="lazy" decoding="async" width="800" height="471" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-05-storage-location.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-05-storage-location.png 962w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-05-storage-location-300x177.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-05-storage-location-768x452.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-05-storage-location-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-05-storage-location-150x88.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

And the internal network which the appliance will be connected to:

<div id='gallery-7' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-06-network.png'><img loading="lazy" decoding="async" width="800" height="469" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-06-network.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-06-network.png 963w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-06-network-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-06-network-768x451.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-06-network-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-06-network-150x88.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Make sure in the 'Customize template' screen (below) you change the 'Deployment Type' to 'cx-cloud-service' and don't leave the default selection (cx-connector) selected as this will install the customer/tenant environment instead of the service provider configuration! The rest of the configuration options on this page are straightforward:

<div id='gallery-8' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-07-customise-ovf.png'><img loading="lazy" decoding="async" width="800" height="675" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-07-customise-ovf.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-07-customise-ovf.png 995w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-07-customise-ovf-300x253.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-07-customise-ovf-768x648.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-07-customise-ovf-178x150.png 178w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-07-customise-ovf-150x126.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

A summary screen is displayed showing a summary of the customization options selected, check these carefully as if they are wrong you'll probably have to re-deploy from scratch:

<div id='gallery-9' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-08-confirm.png'><img loading="lazy" decoding="async" width="800" height="457" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-08-confirm.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-08-confirm.png 1010w, https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-08-confirm-300x171.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-08-confirm-768x439.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-08-confirm-250x143.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-08-confirm-150x86.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Once the appliance is deployed, you will need to manually power it on from the vSphere client (or I did anyway - not sure if this is by design or not). Once it has booted and configured itself it will show the browser link to access to begin the environment configuration:

<div id='gallery-10' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-09-vm-console.png'><img loading="lazy" decoding="async" width="800" height="655" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-09-vm-console.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-09-vm-console.png 801w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-09-vm-console-300x246.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-09-vm-console-768x629.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-09-vm-console-183x150.png 183w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-09-vm-console-150x123.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Note that if you open a page to just the hostname/IP address you'll get an error, you must include the '/ui/mgmt' suffix to the URL. You can now login with the 'initial root login' password you configured during the ova deployment. As you can see from the screen grab below I pre-configured DNS entries for the 3 provider components and used these wherever possible to avoid IP address confusion:

<div id='gallery-11' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/11-deploy-10-login-appliance.png'><img loading="lazy" decoding="async" width="800" height="488" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/11-deploy-10-login-appliance-1024x624.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/11-deploy-10-login-appliance-1024x624.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-deploy-10-login-appliance-300x183.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-deploy-10-login-appliance-768x468.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-deploy-10-login-appliance-246x150.png 246w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-deploy-10-login-appliance-150x91.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-deploy-10-login-appliance.png 1315w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The main screen opens to the Setup Wizard, the tabs at the top of the screen allow you to easily navigate between sections, but these won't show much until you complete the wizard:

<div id='gallery-12' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/12-deploy-11-initial-page.png'><img loading="lazy" decoding="async" width="800" height="481" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/12-deploy-11-initial-page-1024x616.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/12-deploy-11-initial-page-1024x616.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-deploy-11-initial-page-300x181.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-deploy-11-initial-page-768x462.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-deploy-11-initial-page-250x150.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-deploy-11-initial-page-150x90.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-deploy-11-initial-page.png 1331w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Clicking on the 'Setup Wizard' opens a series of dialogs to provide the initial system configuration, first we have to specify the management vCenter authentication details. Note that the 'Lookup Service URL' as well as being optional also requires the path to the Platform Services Controller (PSC) if you are using external PSCs. The full path is truncated in this grab but should be https://<psc or vcenter with embedded psc address>/lookupservice/sdk:

<div id='gallery-13' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/13-setup-01-Management-vCenter.png'><img loading="lazy" decoding="async" width="800" height="481" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/13-setup-01-Management-vCenter-1024x616.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/13-setup-01-Management-vCenter-1024x616.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-setup-01-Management-vCenter-300x180.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-setup-01-Management-vCenter-768x462.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-setup-01-Management-vCenter-250x150.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-setup-01-Management-vCenter-150x90.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-setup-01-Management-vCenter.png 1332w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The wizard includes very useful feedback at each step to show you if the previous actions have been successful or not, just click 'Next' through if everything is ok, or go back and fix the issue if not:

<div id='gallery-14' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/14-setup-02-link-to-mgmt.png'><img loading="lazy" decoding="async" width="800" height="530" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/14-setup-02-link-to-mgmt.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/14-setup-02-link-to-mgmt.png 863w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-setup-02-link-to-mgmt-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-setup-02-link-to-mgmt-768x509.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-setup-02-link-to-mgmt-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-setup-02-link-to-mgmt-150x99.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Now we need to provide a 'system' (administrator) level login to vCloud Director, you don't need to specify the @system part of the user name here:

<div id='gallery-15' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/15-setup-03-vCloud-Director.png'><img loading="lazy" decoding="async" width="800" height="530" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/15-setup-03-vCloud-Director.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/15-setup-03-vCloud-Director.png 862w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-setup-03-vCloud-Director-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-setup-03-vCloud-Director-768x509.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-setup-03-vCloud-Director-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-setup-03-vCloud-Director-150x99.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Again we get confirmation that we've successfully linked to vCloud Director and can continue with 'Next':

<div id='gallery-16' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/16-setup-04-link-to-vCD.png'><img loading="lazy" decoding="async" width="800" height="530" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/16-setup-04-link-to-vCD.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/16-setup-04-link-to-vCD.png 862w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-setup-04-link-to-vCD-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-setup-04-link-to-vCD-768x509.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-setup-04-link-to-vCD-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-setup-04-link-to-vCD-150x99.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next we can add the resource vCenters (where customer workloads actually run). In my lab environment this is the same vCenter that supports the management environment so the details are the same, but in production environments this will almost certainly be different. The setup wizard is intelligent enough to retrieve the names of any vCenter servers being used in Provider VDCs (pVDCs) in vCloud Director so for these you only need to 'Update'.

<div id='gallery-17' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/17-setup-05-resource-vcenters.png'><img loading="lazy" decoding="async" width="800" height="529" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/17-setup-05-resource-vcenters.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/17-setup-05-resource-vcenters.png 863w, https://kiwicloud.ninja/wp-content/uploads/2017/10/17-setup-05-resource-vcenters-300x198.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/17-setup-05-resource-vcenters-768x508.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/17-setup-05-resource-vcenters-227x150.png 227w, https://kiwicloud.ninja/wp-content/uploads/2017/10/17-setup-05-resource-vcenters-150x99.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

When you click update you'll be asked to provide administrator credentials to the resource vCenter environment. Be careful here as the default 'Lookup Service URL' will be set to the vCenter name, even if the vCenter is using an external Platform Services Controller (PSC) as mine was and will need to be manually edited to point to the PSC. This caught me out initially and I couldn't work out why authentication to the resource vCenter was failing.

<div id='gallery-18' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/18-setup-06-resource-vcenter.png'><img loading="lazy" decoding="async" width="577" height="575" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/18-setup-06-resource-vcenter.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/18-setup-06-resource-vcenter.png 577w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-setup-06-resource-vcenter-150x150.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-setup-06-resource-vcenter-300x300.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-setup-06-resource-vcenter-151x150.png 151w" sizes="(max-width: 577px) 100vw, 577px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Once the resource vCenter(s) are authenticated they'll show as 'Registered' in the wizard:

<div id='gallery-19' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/19-setup-07-resource-vcenter-linked.png'><img loading="lazy" decoding="async" width="800" height="529" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/19-setup-07-resource-vcenter-linked.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/19-setup-07-resource-vcenter-linked.png 863w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-setup-07-resource-vcenter-linked-300x198.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-setup-07-resource-vcenter-linked-768x508.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-setup-07-resource-vcenter-linked-227x150.png 227w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-setup-07-resource-vcenter-linked-150x99.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next we need to configure the 2nd appliance configuration - this will be the 'Replication Manager' (also called the Cloud Continuity Manager / CCM in the documentation). We need to specify the parameters shown (the dialog scrolls down and also asks for default gateway address, DNS server address and netmask).

<div id='gallery-20' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/20-setup-08-replication-manager.png'><img loading="lazy" decoding="async" width="800" height="532" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/20-setup-08-replication-manager.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/20-setup-08-replication-manager.png 865w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-setup-08-replication-manager-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-setup-08-replication-manager-768x511.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-setup-08-replication-manager-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-setup-08-replication-manager-150x100.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The wizard will now deploy and start up the replication manager appliance on the vCenter specified. If the networking information is incorrect the process will stall at this point as the wizard relies on establishing network connectivity with the replication manager before continuing. A status update is given at the top of the dialog as the appliance is deployed and started up. Once the replication manager appliance is running and seen on the network you'll see the success message:

<div id='gallery-21' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/22-setup-10-replication-manager-deployed.png'><img loading="lazy" decoding="async" width="800" height="532" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/22-setup-10-replication-manager-deployed.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/22-setup-10-replication-manager-deployed.png 864w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-setup-10-replication-manager-deployed-300x200.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-setup-10-replication-manager-deployed-768x511.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-setup-10-replication-manager-deployed-225x150.png 225w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-setup-10-replication-manager-deployed-150x100.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next the replication manager appliance must be 'activated' by setting the password for the root user and the 'Public Endpoint URL'. Make sure you set this to the correct external (public) IP address that your customers will be using to connect to your CX environment. I haven't found any way yet to alter this setting after deployment if specified incorrectly without deleting the entire CX environment and starting over (the xx's in this grab are simply to hide the real internet addressing I was using - I'm also pretty sure I eventually used the default port of 8044 for this public URL):

<div id='gallery-22' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/23-setup-11-replication-manager-activate.png'><img loading="lazy" decoding="async" width="800" height="531" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/23-setup-11-replication-manager-activate.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/23-setup-11-replication-manager-activate.png 864w, https://kiwicloud.ninja/wp-content/uploads/2017/10/23-setup-11-replication-manager-activate-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/23-setup-11-replication-manager-activate-768x509.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/23-setup-11-replication-manager-activate-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/23-setup-11-replication-manager-activate-150x99.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

If everything has gone ok, you'll get the screen below showing that the replication manager deployment has succeeded and you can move on to the replicator configuration:

<div id='gallery-23' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/24-setup-12-replication-manager-activated.png'><img loading="lazy" decoding="async" width="800" height="531" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/24-setup-12-replication-manager-activated.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/24-setup-12-replication-manager-activated.png 865w, https://kiwicloud.ninja/wp-content/uploads/2017/10/24-setup-12-replication-manager-activated-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/24-setup-12-replication-manager-activated-768x510.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/24-setup-12-replication-manager-activated-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/24-setup-12-replication-manager-activated-150x100.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The deployment details for the Replicator are specified next - the wizard helpfully copies across some of the settings from the Replication Manager deployment, but you still need to specify the (unique) IP and Netmask details:

<div id='gallery-24' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/25-setup-13-replication.png'><img loading="lazy" decoding="async" width="800" height="531" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/25-setup-13-replication.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/25-setup-13-replication.png 862w, https://kiwicloud.ninja/wp-content/uploads/2017/10/25-setup-13-replication-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/25-setup-13-replication-768x510.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/25-setup-13-replication-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/25-setup-13-replication-150x100.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The Replicator appliance will now be deployed in vCenter in exactly the same way as the Replication Manager was previously. Once it becomes available on the network the wizard will detect this and show the screen below:

<div id='gallery-25' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/27-setup-15-replication-activate.png'><img loading="lazy" decoding="async" width="800" height="530" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/27-setup-15-replication-activate.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/27-setup-15-replication-activate.png 866w, https://kiwicloud.ninja/wp-content/uploads/2017/10/27-setup-15-replication-activate-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/27-setup-15-replication-activate-768x509.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/27-setup-15-replication-activate-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/27-setup-15-replication-activate-150x99.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next we have to 'Activate' the Replicator appliance by completing the settings shown below to authenticate to the resource vCenter which this Replicator will be responsible for.

<div id='gallery-26' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/28-setup-16-replication-activate.png'><img loading="lazy" decoding="async" width="800" height="529" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/28-setup-16-replication-activate.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/28-setup-16-replication-activate.png 863w, https://kiwicloud.ninja/wp-content/uploads/2017/10/28-setup-16-replication-activate-300x198.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/28-setup-16-replication-activate-768x508.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/28-setup-16-replication-activate-227x150.png 227w, https://kiwicloud.ninja/wp-content/uploads/2017/10/28-setup-16-replication-activate-150x99.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

If everything worked ok you'll get a 'Successfully Activated' message:

<div id='gallery-27' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/29-setup-17-replicator-activated.png'><img loading="lazy" decoding="async" width="800" height="531" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/29-setup-17-replicator-activated.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/29-setup-17-replicator-activated.png 862w, https://kiwicloud.ninja/wp-content/uploads/2017/10/29-setup-17-replicator-activated-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/29-setup-17-replicator-activated-768x510.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/29-setup-17-replicator-activated-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/29-setup-17-replicator-activated-150x100.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Clicking 'Next' takes you to the 'Complete' screen and shows that if you have additional Resource vCenters you'll need to deploy additional Replicator appliances for these (1 per vCenter):

<div id='gallery-28' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/30-setup-18-completed.png'><img loading="lazy" decoding="async" width="800" height="530" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/30-setup-18-completed.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/30-setup-18-completed.png 865w, https://kiwicloud.ninja/wp-content/uploads/2017/10/30-setup-18-completed-300x199.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/30-setup-18-completed-768x509.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/30-setup-18-completed-226x150.png 226w, https://kiwicloud.ninja/wp-content/uploads/2017/10/30-setup-18-completed-150x99.png 150w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Clicking through the tabs in the management UI should now show that all the required CX components are now deployed and registered. The 'Cloud Resoures' tab shows linked vCloud Director instances and resource vCenters:

<div id='gallery-29' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-01-Cloud-Resources.png'><img loading="lazy" decoding="async" width="800" height="429" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-01-Cloud-Resources-1024x549.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-01-Cloud-Resources-1024x549.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-01-Cloud-Resources-300x161.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-01-Cloud-Resources-768x411.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-01-Cloud-Resources-250x134.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-01-Cloud-Resources-150x80.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-01-Cloud-Resources.png 1217w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The 'Replication Manager' tab shows the deployed Replication Manager appliance:

<div id='gallery-30' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-02-Replication-Manager.png'><img loading="lazy" decoding="async" width="800" height="259" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-02-Replication-Manager-1024x331.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-02-Replication-Manager-1024x331.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-02-Replication-Manager-300x97.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-02-Replication-Manager-768x248.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-02-Replication-Manager-250x81.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-02-Replication-Manager-150x48.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-02-Replication-Manager.png 1207w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Th 'Replicators' tab shows the deployed Replicator appliance(s) - 1 per resource vCenter if you have multiples of these.

<div id='gallery-31' class='gallery galleryid-213 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-03-Replicators.png'><img loading="lazy" decoding="async" width="800" height="274" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-03-Replicators-1024x351.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-03-Replicators-1024x351.png 1024w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-03-Replicators-300x103.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-03-Replicators-768x263.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-03-Replicators-250x86.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-03-Replicators-150x51.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/31-post-setup-03-Replicators.png 1205w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

That completes the appliance installation and initial configuration, next you will need to configure appropriate NAT/firewall rules so that customers on the internet can connect to your new CX service!

Assuming that you wish to use a single external (public) Internet IP address for the entire CX service, the configuration is a little tricky since traffic will need to be directed to either the CX, Replication Manager or Replicator appliance depending on what port it is attempting to aceess. The NAT/Firewall rules that I worked out from the documentation and found that worked are:

<table>
  <tr>
    <td>
      Source Address
    </td>
    
    <td>
      Destination
    </td>
    
    <td>
      Destination Port/Protocol
    </td>
    
    <td>
      Translated Port/Protocol
    </td>
    
    <td>
      Translated Internal Address
    </td>
  </tr>
  
  <tr>
    <td>
      External (Internet)
    </td>
    
    <td>
      CX Service Public IP Address
    </td>
    
    <td>
      443/tcp
    </td>
    
    <td>
      443/tcp
    </td>
    
    <td>
      CX (vCD Extender) appliance internal address
    </td>
  </tr>
  
  <tr>
    <td>
      External (Internet)
    </td>
    
    <td>
      CX Service Public IP Address
    </td>
    
    <td>
      8044/tcp
    </td>
    
    <td>
      8044/tcp
    </td>
    
    <td>
      Replication Manager appliance internal address
    </td>
  </tr>
  
  <tr>
    <td>
      External (Internet)
    </td>
    
    <td>
      CX Service Public IP Address
    </td>
    
    <td>
      44045/tcp
    </td>
    
    <td>
      44045/tcp
    </td>
    
    <td>
      Replicator appliance internal address
    </td>
  </tr>
</table>

Also note that if you restrict outbound internet traffic from your CX network you will also need to permit the following traffic in an Outbound direction:

<table>
  <tr>
    <td>
      Source
    </td>
    
    <td>
      Destination
    </td>
    
    <td>
      Source Port/Protocol
    </td>
    
    <td>
      Destination Port/Protocol
    </td>
    
    <td>
      Description
    </td>
  </tr>
  
  <tr>
    <td>
      CX Server Network
    </td>
    
    <td>
      External (Internet)
    </td>
    
    <td>
      Any
    </td>
    
    <td>
      443/tcp
    </td>
    
    <td>
      Required for CX to be able to communicate with customer Replicator management interface
    </td>
  </tr>
  
  <tr>
    <td>
      CX Server Network
    </td>
    
    <td>
      External (Internet)
    </td>
    
    <td>
      Any
    </td>
    
    <td>
      44045/tcp
    </td>
    
    <td>
      Required for CX to be able to communicate with customer Replicator data interface
    </td>
  </tr>
</table>

In the next part of this series of articles I'll continue with the installation and configuration of the CX components required on the customer / tenant site.

[Link back to Part 1][1] || [Link to Part 3][2]

As always, corrections, comments and feedback are always appreciated.

Jon.

 [1]: http://152.67.105.113/2017/10/vcloud-director-extender-part-1-overview/
 [2]: http://152.67.105.113/2017/10/vcloud-director-extender-part-3-tenant-setup/