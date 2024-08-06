---
title: vCloud Director Extender – Part 5 – Stretch Networking (L2VPN)
author: Jon Waite
type: post
date: 2017-11-25T04:16:12+00:00
url: /2017/11/vcloud-director-extender-part-5-stretch-networking-l2vpn/
categories:
  - Networking
  - vCloud Director
  - vCloud Director Extender
  - VMware
tags:
  - CX
  - Installation
  - Networking
  - NSX
  - vCloud Director
  - vCloud Director 9
  - vCloud Director Extender
  - VMware
  - vSphere 6

---
In this 5th part of my look into vCloud Director Extender (CX), I deal with the extension of a customer vCenter network into a cloud provider network using the L2VPN network extension functionality. Apologies that this post has been a bit delayed, turned out that I needed a VMware support request and a code update to vCloud Director 9.0.0.1 before I could get this functionality working. (I also had an issue with my lab environment which runs as a nested platform inside a vCloud Director environment and it turned out that the networking environment I had wasn&#8217;t quite flexible enough to get this working).

**Update:** an earlier version of this article didn&#8217;t include the steps to configure the L2 appliance settings in the vCloud Director Extender web interface &#8211; I&#8217;ve now added these to provide a more complete guide.

Links to the other parts of this series:  
[Part 1 &#8211; Overview][1]  
[Part 2 &#8211; Cloud Provider / Service Provider installation and configuration (MyCloud)][2]  
[Part 3 &#8211; Customer / Tenant installation and configuration (Tyrell)][3]  
[Part 4 &#8211; Customer / Tenant connecting to a Cloud Provider and Virtual Machine migration (Tyrell)][4]

I won&#8217;t deal with the use-case here that the customer already has NSX networking installed and configured, since in most cases you can simply create L2VPN networks directly between the customer and provider NSX Edge appliances and don&#8217;t really need to use the CX L2VPN functionality.

In order to be able to use the standalone L2VPN connectivity, the following pre-requisites are required:

  * A tenant vSphere environment with the vCloud Director Extender appliance deployed (it does not appear to be necessary to deploy the replication appliance if you only wish to use the L2VPN functionality, but obviously if you are intending to migrate VMs too you will need this deployed and configured as described in [Part 3][3] of this series. In either case you will still need to register the cloud provider in the CX interface.
  * A configured vCloud Director VDC for the tenant to connect to. This environment must also have an Advanced Edge Gateway deployed with at least one uplink having a publicly accessible (internet) IP address. Note that you do not need to configure the L2VPN service on this gateway &#8211; the CX wizard completes this for you.
  * At least one OrgVDC network created as a subinterface on this edge gateway. The steps to create a suitable new OrgVDC network are detailed below.
  * Outbound internet connectivity to allow the standalone edge deployed in the tenant vCenter to communicate with the cloud-hosted edge gateway &#8211; only port 443/tcp is required for this.
  * Administrative credentials to connect to both the tenant vCenter and the cloud tenancy/VDC (Organization Administrator role is required).

Opening the tenant vCenter environment and selecting the &#8216;Home&#8217; page shows the following:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-416" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/0501-vSphere-Web-Client-Start-800x472.png" alt="" width="800" height="472" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/0501-vSphere-Web-Client-Start-800x472.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0501-vSphere-Web-Client-Start-300x177.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0501-vSphere-Web-Client-Start-768x453.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0501-vSphere-Web-Client-Start-250x147.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0501-vSphere-Web-Client-Start-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0501-vSphere-Web-Client-Start.png 1145w" sizes="(max-width: 800px) 100vw, 800px" />][5]

Selecting the vCloud Director Extender icon opens the CX interface:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-417" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/0502-Provider-Clouds-registered-800x225.png" alt="" width="800" height="225" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/0502-Provider-Clouds-registered-800x225.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0502-Provider-Clouds-registered-300x84.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0502-Provider-Clouds-registered-768x216.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0502-Provider-Clouds-registered-250x70.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0502-Provider-Clouds-registered-150x42.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0502-Provider-Clouds-registered.png 1303w" sizes="(max-width: 800px) 100vw, 800px" />][6]

If you have not yet configured the L2 appliance settings, selecting the &#8216;DC Extensions&#8217; tab will show the following error:  
[  
<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-418" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/0503-Error-DC-Extension.png" alt="" width="579" height="207" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/0503-Error-DC-Extension.png 579w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0503-Error-DC-Extension-300x107.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0503-Error-DC-Extension-250x89.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0503-Error-DC-Extension-150x54.png 150w" sizes="(max-width: 579px) 100vw, 579px" />][7] 

To fix this, open the vCloud Director Extender web interface in a browser by opening https://<ip address of deployed cx appliance>/ and log in, select the &#8216;DC Extensions&#8217; tab:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-419" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/0504-Adding-L2-appliance-config-01-800x349.png" alt="" width="800" height="349" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/0504-Adding-L2-appliance-config-01-800x349.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0504-Adding-L2-appliance-config-01-300x131.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0504-Adding-L2-appliance-config-01-768x335.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0504-Adding-L2-appliance-config-01-250x109.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0504-Adding-L2-appliance-config-01-150x65.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0504-Adding-L2-appliance-config-01.png 1352w" sizes="(max-width: 800px) 100vw, 800px" />][8]

Select the &#8216;Add Appliance Configuration&#8217; option and complete the form to provide the deployment parameters where the standalone NSX edge appliance will be deployed:

[<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-421" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/0506-Adding-L2-appliance-config-03.png" alt="" width="580" height="604" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/0506-Adding-L2-appliance-config-03.png 580w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0506-Adding-L2-appliance-config-03-288x300.png 288w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0506-Adding-L2-appliance-config-03-144x150.png 144w" sizes="(max-width: 580px) 100vw, 580px" />][9]

The &#8216;Uplink Network Pool IP&#8217; setting is a bit strange &#8211; it appears to be asking for a network pool or IP range, but the &#8216;help text&#8217; in the field is asking for a single IP address. I found that the validation on this field is a bit odd &#8211; it will basically accept any input at all (even random strings) without complaining, but obviously deployment won&#8217;t work. What you need to do is add individual IPv4 addresses and click the &#8216;Add&#8217; button for each. You will need 1 address for each stretched network you will be extending to your cloud platform. In this example I am only extending a single network so have added a single IPv4 address (192.168.0.201).

Once you click the &#8216;Create&#8217; button you will be returned to the &#8216;DC Extensions&#8217; tab and shown a summary of the L2 appliance configuration:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-422" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/0507-Added-L2-appliance-config-04-800x518.png" alt="" width="800" height="518" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/0507-Added-L2-appliance-config-04-800x518.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0507-Added-L2-appliance-config-04-300x194.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0507-Added-L2-appliance-config-04-768x498.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0507-Added-L2-appliance-config-04-232x150.png 232w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0507-Added-L2-appliance-config-04-150x97.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/0507-Added-L2-appliance-config-04.png 1201w" sizes="(max-width: 800px) 100vw, 800px" />][10]

Note that there doesn&#8217;t appear to be any way to edit an existing L2 Appliance configuration, so if you need to change settings (e.g. to add additional uplink IP pool addresses) you will likely need to delete and recreate the entire entry.

&nbsp;

Next we need to add a new &#8216;subinterface&#8217; network to our hosted Edge gateway appliance, logging in to our cloud provider portal we can select the &#8216;Administration&#8217; tab and the &#8216;Org VDC Networks&#8217; sub-option, clicking the &#8216;Add&#8217; button shows the dialog to create a new Org VDC Network. We need to select &#8216;Create a routed network by connecting to an existing edge gateway&#8217; and then check the &#8216;Create as subinterface&#8217; check box:

<div id='gallery-55' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/01-OrgVDC-Network-01.png'><img loading="lazy" decoding="async" width="800" height="707" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/01-OrgVDC-Network-01-800x707.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/01-OrgVDC-Network-01-800x707.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/01-OrgVDC-Network-01-300x265.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/01-OrgVDC-Network-01-768x679.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/01-OrgVDC-Network-01-170x150.png 170w, https://kiwicloud.ninja/wp-content/uploads/2017/11/01-OrgVDC-Network-01-150x133.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/01-OrgVDC-Network-01.png 850w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next we configure the standard network information (Gateway, Network mask, DNS etc.) Since this network will be bridged to our on-premises network we can use the same details. Optionally a new Static IP pool can also be created so that new VMs provisioned in the cloud service can use this pool for their IP addresses. This won&#8217;t be an issue for VMs being migrated as they will carry across whatever IP addresses are already assigned to them. Note that the gateway address is set to be the same address as the existing (on-premises) gateway &#8211; this means that re-configuring the default gateway setting in the guest OS isn&#8217;t required either:

<div id='gallery-56' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/02-OrgVDC-Network-02.png'><img loading="lazy" decoding="async" width="800" height="707" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/02-OrgVDC-Network-02-800x707.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/02-OrgVDC-Network-02-800x707.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/02-OrgVDC-Network-02-300x265.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/02-OrgVDC-Network-02-768x679.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/02-OrgVDC-Network-02-170x150.png 170w, https://kiwicloud.ninja/wp-content/uploads/2017/11/02-OrgVDC-Network-02-150x133.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/02-OrgVDC-Network-02.png 850w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Now we supply a name for the new Org VDC network and optionally a description. The check box can also be used if the customer has multiple VDCs and wishes to share the new network across them:

<div id='gallery-57' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/03-OrgVDC-Network-03.png'><img loading="lazy" decoding="async" width="800" height="707" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/03-OrgVDC-Network-03-800x707.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/03-OrgVDC-Network-03-800x707.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/03-OrgVDC-Network-03-300x265.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/03-OrgVDC-Network-03-768x679.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/03-OrgVDC-Network-03-170x150.png 170w, https://kiwicloud.ninja/wp-content/uploads/2017/11/03-OrgVDC-Network-03-150x133.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/03-OrgVDC-Network-03.png 850w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Finally the summary screen allows us to check the information provided and go back and make any changes required if not correct. The most important setting is to make sure the network is attached to the edge gateway as a subinterface:

<div id='gallery-58' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/04-OrgVDC-Network-04.png'><img loading="lazy" decoding="async" width="800" height="707" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/04-OrgVDC-Network-04-800x707.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/04-OrgVDC-Network-04-800x707.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/04-OrgVDC-Network-04-300x265.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/04-OrgVDC-Network-04-768x679.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/04-OrgVDC-Network-04-170x150.png 170w, https://kiwicloud.ninja/wp-content/uploads/2017/11/04-OrgVDC-Network-04-150x133.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/04-OrgVDC-Network-04.png 850w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Once finished creating, the Org VDC network will be shown in the list with a type of &#8216;Routed&#8217; and an interface type of &#8216;Subinterface&#8217;:

<div id='gallery-59' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/05-Edge-Gateway-with-SI.png'><img loading="lazy" decoding="async" width="800" height="41" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/05-Edge-Gateway-with-SI-800x41.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/05-Edge-Gateway-with-SI-800x41.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/05-Edge-Gateway-with-SI-300x15.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/05-Edge-Gateway-with-SI-768x39.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/05-Edge-Gateway-with-SI-250x13.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/05-Edge-Gateway-with-SI-150x8.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/05-Edge-Gateway-with-SI.png 1037w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Next we access the vCloud Extender interface from within the customer vCenter plugin, selecting the &#8216;DC Extensions&#8217; tab takes us to the following dialog:

<div id='gallery-60' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/06-Add-DC-Extension-Net-01.png'><img loading="lazy" decoding="async" width="800" height="278" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/06-Add-DC-Extension-Net-01-800x278.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/06-Add-DC-Extension-Net-01-800x278.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/06-Add-DC-Extension-Net-01-300x104.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/06-Add-DC-Extension-Net-01-768x266.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/06-Add-DC-Extension-Net-01-250x87.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/06-Add-DC-Extension-Net-01-150x52.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/06-Add-DC-Extension-Net-01.png 1199w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Selecting &#8216;New Extension&#8217; shows the dialog to create a new L2 extension, the fields are mostly populated for you. The &#8216;Enable egress&#8217; allows you to select which gateway(s) will be allowed to forward traffic outside of the extended network. In this example I&#8217;ve only configured egress on the Source (on-premises) side through the existing gateway:

<div id='gallery-61' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/07-Add-DC-Extension-Net-02.png'><img loading="lazy" decoding="async" width="800" height="595" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/07-Add-DC-Extension-Net-02-800x595.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/07-Add-DC-Extension-Net-02-800x595.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/07-Add-DC-Extension-Net-02-300x223.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/07-Add-DC-Extension-Net-02-768x571.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/07-Add-DC-Extension-Net-02-202x150.png 202w, https://kiwicloud.ninja/wp-content/uploads/2017/11/07-Add-DC-Extension-Net-02-150x112.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/07-Add-DC-Extension-Net-02.png 1198w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

When you click &#8216;Start&#8217;, the status will go to &#8216;Connecting&#8217; and a number of activities will take place in the customer vCenter:

<div id='gallery-62' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/08-Add-DC-Extension-Net-03.png'><img loading="lazy" decoding="async" width="800" height="243" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/08-Add-DC-Extension-Net-03-800x243.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/08-Add-DC-Extension-Net-03-800x243.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/08-Add-DC-Extension-Net-03-300x91.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/08-Add-DC-Extension-Net-03-768x233.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/08-Add-DC-Extension-Net-03-250x76.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/08-Add-DC-Extension-Net-03-150x46.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/08-Add-DC-Extension-Net-03.png 1198w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Reading from the bottom (oldest) upwards, a new port group is created, an NSX Edge Standalone appliance is deployed and powered-on and the new port group is reconfigured once this has completed (ignore the VM migration task, that just happened to occur during the same time window in my lab). In this case the new NSX standalone edge was named &#8216;mcloudext-edge-4&#8217; and the port group &#8216;mcxt-tpg-l2vpn-vlan-Tyrell-VDC15&#8217;.

<div id='gallery-63' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/10-DC-Extension-VC-Tasks.png'><img loading="lazy" decoding="async" width="800" height="127" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/10-DC-Extension-VC-Tasks-800x127.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/10-DC-Extension-VC-Tasks-800x127.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/10-DC-Extension-VC-Tasks-300x48.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/10-DC-Extension-VC-Tasks-768x122.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/10-DC-Extension-VC-Tasks-250x40.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/10-DC-Extension-VC-Tasks-150x24.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/10-DC-Extension-VC-Tasks.png 1427w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

Once deployment has completed (takes a few minutes) the vCloud Extender client interface shows the new DC extension network with a status of &#8216;Connected&#8217;:

<div id='gallery-64' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/09-Add-DC-Extension-Net-04.png'><img loading="lazy" decoding="async" width="800" height="252" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/09-Add-DC-Extension-Net-04-800x252.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/09-Add-DC-Extension-Net-04-800x252.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/09-Add-DC-Extension-Net-04-300x95.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/09-Add-DC-Extension-Net-04-768x242.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/09-Add-DC-Extension-Net-04-250x79.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/09-Add-DC-Extension-Net-04-150x47.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/09-Add-DC-Extension-Net-04.png 1198w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

In the tenant vCloud Director portal you can also see the status of the tunnel under &#8216;Statistics&#8217; and &#8216;L2 VPN&#8217; within the edge gateway interface:

<div id='gallery-65' class='gallery galleryid-399 gallery-columns-1 gallery-size-large'>
  <dl class='gallery-item'>
    <dt class='gallery-icon landscape'>
      <a href='https://kiwicloud.ninja/wp-content/uploads/2017/11/11-vCD-L2VPN-Stats.png'><img loading="lazy" decoding="async" width="800" height="266" src="https://kiwicloud.ninja/wp-content/uploads/2017/11/11-vCD-L2VPN-Stats-800x266.png" class="attachment-large size-large" alt="" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/11/11-vCD-L2VPN-Stats-800x266.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/11/11-vCD-L2VPN-Stats-300x100.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/11/11-vCD-L2VPN-Stats-768x256.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/11/11-vCD-L2VPN-Stats-250x83.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/11/11-vCD-L2VPN-Stats-150x50.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/11/11-vCD-L2VPN-Stats.png 1064w" sizes="(max-width: 800px) 100vw, 800px" /></a>
    </dt>
  </dl>
  
  <br style="clear: both" />
</div>

You will now find that any VMs connected to the stretched network (OrgVDC network) in your cloud environment have L2 connectivity with the on-premises network and will continue to function as if they were still located in the customer&#8217;s own datacenter.

As I mentioned at the start of this post, I hit a number of issues when configuring this environment and getting it working took several attempts and a couple of rebuilds of my lab. The main issue was that in the initial release of vCloud Director v9.0.0.0 there is an issue that prevents the details required for the standalone NSX edge being deployed from being returned by the API. This prevents the deployment of the customer edge at all and resulted in my VMware support call. The specific issue is referenced in the vCloud Director 9.0.0.1 [release notes][11]{.broken_link}  as &#8216;Resolves an issue where the vCloud Director API does not return a tunnelID parameter in response to a GET /vdcnetworks request sent against a routed Organization VCD network that has a subinterface enabled.&#8217; As far as I can work out, it will be impossible to successfully use L2VPN in CX without upgrading the provider to vCloud Director 9.0.0.1 to resolve this issue.

The other issue I hit in my lab was that my hosted &#8216;Tenant Edge&#8217; was NAT&#8217;d behind another NSX Edge gateway which was also performing NAT translation (Double-NAT). This was due to the way my lab is built in a nested environment inside vCloud Director. Unfortunately this meant the external interface of my hosted &#8216;Tenant Edge&#8217; was actually an internal network address, so when the customer/on-premise edge tried to establish contact it was using an internal network address which obviously wasn&#8217;t going to work. I solved this by connecting a &#8216;real&#8217; external internet network to my hosted Tenant Edge.

As always, comments and feedback always appreciated.

Jon.

 [1]: http://152.67.105.113/2017/10/vcloud-director-extender-part-1-overview/
 [2]: http://152.67.105.113/2017/10/vcloud-director-extender-part-2-cloud-provider-setup/
 [3]: http://152.67.105.113/2017/10/vcloud-director-extender-part-3-tenant-setup/
 [4]: http://152.67.105.113/2017/10/vcloud-director-extender-part-4-connect-to-provider-vm-migration/
 [5]: https://kiwicloud.ninja/wp-content/uploads/2017/11/0501-vSphere-Web-Client-Start.png
 [6]: https://kiwicloud.ninja/wp-content/uploads/2017/11/0502-Provider-Clouds-registered.png
 [7]: https://kiwicloud.ninja/wp-content/uploads/2017/11/0503-Error-DC-Extension.png
 [8]: https://kiwicloud.ninja/wp-content/uploads/2017/11/0504-Adding-L2-appliance-config-01.png
 [9]: https://kiwicloud.ninja/wp-content/uploads/2017/11/0506-Adding-L2-appliance-config-03.png
 [10]: https://kiwicloud.ninja/wp-content/uploads/2017/11/0507-Added-L2-appliance-config-04.png
 [11]: https://docs.vmware.com/en/VMware-vCloud-Director-for-Service-Providers/9.0.0.1/rn/rel_notes_vcloud_director_9-0-0-1.html