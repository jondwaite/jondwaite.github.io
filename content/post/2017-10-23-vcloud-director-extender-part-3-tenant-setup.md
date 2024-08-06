---
title: vCloud Director Extender – Part 3 – Tenant Setup
author: Jon Waite
type: post
date: 2017-10-22T23:30:57+00:00
url: /2017/10/vcloud-director-extender-part-3-tenant-setup/
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
In [part 1][1] and [part 2][2] of this series I detailed an overview of VMware vCloud Director Extender (CX) and the configuration from a Service Provider perspective to configure their platform to support CX.

This third article in the series details the configuration steps required for a tenant/customer environment to deploy and configure CX into their environment.

Once a service provider configuration is complete, any customers of that provider with sufficient allocated resources in a Virtual Datacenter (VDC) can configure the tenant CX environment and connect this to their vCenter environment. Once complete they will be able to migrate and replicate vSphere VMs between their own vCenter and the service provider datacenter extremely easily. Optionally they can use L2VPN functionality to stretch their networks into the Cloud Provider&#8217;s datacenter removing the requirement to have a pre-configured network in place. Of course many customers will wish to move to dedicated networking later, but having the initial ability to quickly provision their networks into a Cloud provider can dramatically shorten migration timeframes.

The initial deployment steps for customers deploying CX are exactly the same as for a Service Provider &#8211; download (or have provided to them by their Cloud Provider) the ova appliance for vCloud Director Extender and deploy this into their vCenter environment.

<div id='gallery-32' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCenter-prior-intro.png'><img loading="lazy" decoding="async" width="800" height="463" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCenter-prior-intro-800x463.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCenter-prior-intro-800x463.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCenter-prior-intro-300x174.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCenter-prior-intro-768x444.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCenter-prior-intro-250x145.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCenter-prior-intro-150x87.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCenter-prior-intro.png 1317w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Right-clicking on the desired location and selecting &#8216;Deploy OVF Template&#8230;&#8217; allows the local CX .ova file to be selected

<div id='gallery-33' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/02-select-ova-to-deploy.png'><img loading="lazy" decoding="async" width="800" height="468" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/02-select-ova-to-deploy-800x468.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/02-select-ova-to-deploy-800x468.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-select-ova-to-deploy-300x175.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-select-ova-to-deploy-768x449.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-select-ova-to-deploy-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-select-ova-to-deploy-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-select-ova-to-deploy.png 960w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The appliance name and folder are selected next:

<div id='gallery-34' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-name-and-location.png'><img loading="lazy" decoding="async" width="800" height="467" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-name-and-location-800x467.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-name-and-location-800x467.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-name-and-location-300x175.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-name-and-location-768x448.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-name-and-location-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-name-and-location-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/03-deploy-name-and-location.png 960w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Followed by the vCenter Cluster which will run the deployed appliance:

<div id='gallery-35' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-resource.png'><img loading="lazy" decoding="async" width="800" height="466" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-resource-800x466.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-resource-800x466.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-resource-300x175.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-resource-768x447.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-resource-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-resource-150x87.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-deploy-resource.png 958w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Check the template details and then click &#8216;Next&#8217; to continue:

<div id='gallery-36' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-summary.png'><img loading="lazy" decoding="async" width="800" height="467" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-summary-800x467.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-summary-800x467.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-summary-300x175.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-summary-768x448.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-summary-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-summary-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-deploy-summary.png 961w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Read and accept the VMware license agreement:

<div id='gallery-37' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-license.png'><img loading="lazy" decoding="async" width="800" height="466" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-license-800x466.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-license-800x466.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-license-300x175.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-license-768x448.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-license-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-license-150x87.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-deploy-license.png 961w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next select the Datastore storage on which the appliance will be deployed:

<div id='gallery-38' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-storage.png'><img loading="lazy" decoding="async" width="800" height="467" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-storage-800x467.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-storage-800x467.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-storage-300x175.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-storage-768x448.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-storage-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-storage-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-deploy-storage.png 960w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Select the required network for the appliance:

<div id='gallery-39' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-network.png'><img loading="lazy" decoding="async" width="800" height="468" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-network-800x468.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-network-800x468.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-network-300x175.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-network-768x449.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-network-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-network-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-deploy-network.png 959w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Make sure that &#8216;cx-connector&#8217; (default) is selected for the &#8216;Deployment Type&#8217; and fill out the IP addressing information for the appliance:

<div id='gallery-40' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-network-config.png'><img loading="lazy" decoding="async" width="800" height="467" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-network-config-800x467.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-network-config-800x467.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-network-config-300x175.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-network-config-768x448.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-network-config-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-network-config-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/09-deploy-network-config.png 960w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Check the summary information carefully and click &#8216;Finish&#8217; to begin the deployment operation:

<div id='gallery-41' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-summary.png'><img loading="lazy" decoding="async" width="800" height="466" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-summary-800x466.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-summary-800x466.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-summary-300x175.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-summary-768x448.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-summary-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-summary-150x87.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-deploy-summary.png 961w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Once the appliance deployment task has configured, power-on the deployed VM in vCenter and wait for it to initialise. When it is running you can open a web browser to the IP address you configured for the appliance and login using the password configured. Note that you have to add &#8216;/ui/mgmt&#8217; to the login URL for the appliance, so the full URL will be &#8216;https://<IP address of appliance>/ui/mgmt&#8217;:

<div id='gallery-42' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/11-config-01-login.png'><img loading="lazy" decoding="async" width="800" height="520" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/11-config-01-login-800x520.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/11-config-01-login-800x520.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-config-01-login-300x195.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-config-01-login-768x499.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-config-01-login-231x150.png 231w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-config-01-login-150x98.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-config-01-login.png 1535w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The initial CX dialog when logged in allows you to start the Setup Wizard, note that in contrast to the Service Provider UI, there is no &#8216;Replication Managers&#8217; tab in the cx-connector configuration:

<div id='gallery-43' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/12-config-02-wizard.png'><img loading="lazy" decoding="async" width="800" height="401" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/12-config-02-wizard-800x401.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/12-config-02-wizard-800x401.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-config-02-wizard-300x150.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-config-02-wizard-768x385.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-config-02-wizard-250x125.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-config-02-wizard-150x75.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-config-02-wizard.png 1295w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The first step of the wizard is to link to the existing on-premise vCenter environment, note that if you are using an external Platform Services Controller (PSC) you will need to specify the PSC URL for the Lookup Service URL (although this is optional). The user specified needs to have administrative permissions within the vCenter environment:

<div id='gallery-44' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/13-config-03-onpremise-vcenter.png'><img loading="lazy" decoding="async" width="800" height="666" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/13-config-03-onpremise-vcenter-800x666.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/13-config-03-onpremise-vcenter-800x666.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-config-03-onpremise-vcenter-300x250.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-config-03-onpremise-vcenter-768x640.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-config-03-onpremise-vcenter-180x150.png 180w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-config-03-onpremise-vcenter-150x125.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/13-config-03-onpremise-vcenter.png 862w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Once the vCenter details and credentials are accepted, CX will provide a success notification, click &#8216;Next&#8217; to continue:

<div id='gallery-45' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/14-config-04-onpremise-vcenter-linked.png'><img loading="lazy" decoding="async" width="800" height="665" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/14-config-04-onpremise-vcenter-linked-800x665.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/14-config-04-onpremise-vcenter-linked-800x665.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-config-04-onpremise-vcenter-linked-300x249.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-config-04-onpremise-vcenter-linked-768x638.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-config-04-onpremise-vcenter-linked-181x150.png 181w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-config-04-onpremise-vcenter-linked-150x125.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-config-04-onpremise-vcenter-linked.png 864w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The next page asks you to register the CX plugin with vCenter, this will likely become important in future as CX is updated, but for now leave the Version as 1.0.0 and click &#8216;Next&#8217;:

<div id='gallery-46' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/15-config-05-vcenter-plugin.png'><img loading="lazy" decoding="async" width="800" height="667" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/15-config-05-vcenter-plugin-800x667.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/15-config-05-vcenter-plugin-800x667.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-config-05-vcenter-plugin-300x250.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-config-05-vcenter-plugin-768x641.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-config-05-vcenter-plugin-180x150.png 180w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-config-05-vcenter-plugin-150x125.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-config-05-vcenter-plugin.png 862w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Once the plugin has registered into vCenter you will see a success notification. In testing I found that if the CX plugin had previously been registered with the vCenter (and not manually removed), this step would generate an error notification, but it was still possible to continue with the wizard and everything appeared to function fine afterwards:

<div id='gallery-47' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/16-config-06-vcenter-plugin-done.png'><img loading="lazy" decoding="async" width="800" height="666" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/16-config-06-vcenter-plugin-done-800x666.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/16-config-06-vcenter-plugin-done-800x666.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-config-06-vcenter-plugin-done-300x250.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-config-06-vcenter-plugin-done-768x639.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-config-06-vcenter-plugin-done-180x150.png 180w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-config-06-vcenter-plugin-done-150x125.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-config-06-vcenter-plugin-done.png 863w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next you need to provide the configuration for the &#8216;Replicator&#8217; appliance that will be deployed into the on-premise vCenter. The VMware documentation advises not to use DHCP for this and to manually specify a static IP configuration:

<div id='gallery-48' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/17-config-07-replicator-config.png'><img loading="lazy" decoding="async" width="800" height="664" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/17-config-07-replicator-config-800x664.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/17-config-07-replicator-config-800x664.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/17-config-07-replicator-config-300x249.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/17-config-07-replicator-config-768x638.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/17-config-07-replicator-config-181x150.png 181w, https://kiwicloud.ninja/wp-content/uploads/2017/10/17-config-07-replicator-config-150x125.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/17-config-07-replicator-config.png 850w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The &#8216;Replicator&#8217; appliance is now deployed into vCenter and powered on. Once it has established network communication with the CX environment you will see a success notification:

<div id='gallery-49' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/18-config-08-replicator-done.png'><img loading="lazy" decoding="async" width="800" height="665" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/18-config-08-replicator-done-800x665.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/18-config-08-replicator-done-800x665.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-config-08-replicator-done-300x249.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-config-08-replicator-done-768x638.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-config-08-replicator-done-180x150.png 180w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-config-08-replicator-done-150x125.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-config-08-replicator-done.png 848w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The next step is to activate the Replicator appliance by providing a root password and authentication details for the on-premise vCenter environment. Note that you will need to set the Public Endpoint URL correctly in order for the appliance to be reachable by your cloud provider. If the on-premise Replicator appliance is behind a corporate firewall (as most will be), you will need to configure inbound firewall and translation rules and make sure this field is set correctly.

In my lab setup I configured the replicator public URL to be on port 443 on the public (Internet) address of the outside of the Tyrell firewall and used NAT port translation (see the networking configuration information below).

<div id='gallery-50' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/19-config-09-replicator-activate.png'><img loading="lazy" decoding="async" width="800" height="668" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/19-config-09-replicator-activate-800x668.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/19-config-09-replicator-activate-800x668.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-config-09-replicator-activate-300x251.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-config-09-replicator-activate-768x642.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-config-09-replicator-activate-180x150.png 180w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-config-09-replicator-activate-150x125.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-config-09-replicator-activate.png 863w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

If everything is accepted you&#8217;ll receive a success notification in the wizard (note that I blanked the Public Endpoint URL field in this capture which is why it doesn&#8217;t show in the grab below):

<div id='gallery-51' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/20-config-10-replicator-actived.png'><img loading="lazy" decoding="async" width="800" height="666" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/20-config-10-replicator-actived-800x666.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/20-config-10-replicator-actived-800x666.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-config-10-replicator-actived-300x250.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-config-10-replicator-actived-768x640.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-config-10-replicator-actived-180x150.png 180w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-config-10-replicator-actived-150x125.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-config-10-replicator-actived.png 861w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The wizard is now complete, click &#8216;Finish&#8217; to return to the UX interface:

<div id='gallery-52' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/21-config-11-initial-config-done.png'><img loading="lazy" decoding="async" width="800" height="666" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/21-config-11-initial-config-done-800x666.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/21-config-11-initial-config-done-800x666.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-config-11-initial-config-done-300x250.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-config-11-initial-config-done-768x639.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-config-11-initial-config-done-180x150.png 180w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-config-11-initial-config-done-150x125.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-config-11-initial-config-done.png 864w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The &#8216;vCenter Management&#8217; tab should now show the on-premise vCenter details

<div id='gallery-53' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/23-vcenter-management-tab-after-initial-config.png'><img loading="lazy" decoding="async" width="800" height="301" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/23-vcenter-management-tab-after-initial-config-800x301.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/23-vcenter-management-tab-after-initial-config-800x301.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/23-vcenter-management-tab-after-initial-config-300x113.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/23-vcenter-management-tab-after-initial-config-768x289.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/23-vcenter-management-tab-after-initial-config-250x94.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/23-vcenter-management-tab-after-initial-config-150x57.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/23-vcenter-management-tab-after-initial-config.png 1322w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

The &#8216;Replicators&#8217; tab should show the details for the replicator appliance deployed in the wizard:

<div id='gallery-54' class='gallery galleryid-303 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/10/22-replicators-tab-after-initial-config.png'><img loading="lazy" decoding="async" width="800" height="251" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/22-replicators-tab-after-initial-config-800x251.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/22-replicators-tab-after-initial-config-800x250.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-replicators-tab-after-initial-config-300x94.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-replicators-tab-after-initial-config-768x241.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-replicators-tab-after-initial-config-250x79.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-replicators-tab-after-initial-config-150x47.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-replicators-tab-after-initial-config.png 1318w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Once vCenter has been closed and restarted you should now see a new &#8216;vCloud Director Extender&#8217; item in the UI:

[<img loading="lazy" decoding="async" class="aligncenter wp-image-305 size-large" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/27-vCenter-with-new-vCDE-option-800x463.png" alt="" width="800" height="463" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/27-vCenter-with-new-vCDE-option-800x463.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/27-vCenter-with-new-vCDE-option-300x174.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/27-vCenter-with-new-vCDE-option-768x444.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/27-vCenter-with-new-vCDE-option-250x145.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/27-vCenter-with-new-vCDE-option-150x87.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/27-vCenter-with-new-vCDE-option.png 1317w" sizes="(max-width: 800px) 100vw, 800px" />][3]

The networking configuration for a customer environment is a little simpler than for the cloud provider side, you will need to permit 2 inbound ports through the firewall, both of which need to communicate directly with the &#8216;Replicator&#8217; appliance.

Assuming that you configured the &#8216;Public Endpoint URL&#8217; with port 443, you will need to use NAT translation to divert this to port 8043 on the appliance:

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
      Public IP Address
    </td>
    
    <td>
      443/tcp
    </td>
    
    <td>
      8043/tcp
    </td>
    
    <td>
      Replicator appliance internal address
    </td>
  </tr>
  
  <tr>
    <td>
      External (Internet)
    </td>
    
    <td>
      Public IP Address
    </td>
    
    <td>
      44045/tcp
    </td>
    
    <td>
      44045/tcp
    </td>
    
    <td>
      Replication appliance internal address
    </td>
  </tr>
</table>

You can (and should) limit the public/external addresses permitted to communicate with your Replicator appliance to just those public IP addresses used by your Cloud Provider &#8211; they should be able to provide you with this information.

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
      Cloud Provider Public CX Address
    </td>
    
    <td>
      Any
    </td>
    
    <td>
      443/tcp
    </td>
    
    <td>
      Required for communications with the provider CX appliance
    </td>
  </tr>
  
  <tr>
    <td>
      CX Server Network
    </td>
    
    <td>
      Cloud Provider Public CX Address
    </td>
    
    <td>
      Any
    </td>
    
    <td>
      8044/tcp
    </td>
    
    <td>
      Required for communications with the provider Replication Manager appliance
    </td>
  </tr>
  
  <tr>
    <td>
      CX Server Network
    </td>
    
    <td>
      Cloud Provider Public CX Address
    </td>
    
    <td>
      Any
    </td>
    
    <td>
      44045/tcp
    </td>
    
    <td>
      Required for communications with the provider Replicator appliance
    </td>
  </tr>
</table>

Of course if your provider has configured different ports for these components you will need to allow access to these instead of the defaults listed.

In the next part of this series I&#8217;ll continue with configuring the customer environment to connect to a cloud provider CX environment and to migrate some VMs.

[Link back to Part 2][2] || [Link to Part 4][4]

As always, corrections, comments and feedback are always appreciated.

Jon.

 [1]: http://152.67.105.113/2017/10/vcloud-director-extender-part-1-overview/
 [2]: http://152.67.105.113/2017/10/vcloud-director-extender-part-2-cloud-provider-setup/
 [3]: https://kiwicloud.ninja/wp-content/uploads/2017/10/27-vCenter-with-new-vCDE-option.png
 [4]: http://152.67.105.113/2017/10/vcloud-director-extender-part-4-connect-to-provider-vm-migration/