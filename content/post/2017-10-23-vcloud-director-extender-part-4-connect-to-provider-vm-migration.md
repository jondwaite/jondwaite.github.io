---
title: 'vCloud Director Extender – Part 4 – Connect to Provider & VM Migration'
author: Jon Waite
type: post
date: 2017-10-23T03:51:45+00:00
url: /2017/10/vcloud-director-extender-part-4-connect-to-provider-vm-migration/
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
In the first 3 parts of this series I covered an overview of vCloud Director Extender (CX), the installation and configuration of CX at the Cloud Provider site and the installation and configuration of CX at the customer/tenant site. In this 4th part I will be covering the configuration of the tenant environment to connect to the provider cloud and then migrate VM workloads to the provider.

This part follows on from the configuration completed in part 3 of this series and assumes that Tyrell (the customer site) have an existing virtual datacenter (VDC) environment available from MyCloud (the provider) and an appropriate Organization Administrator login to this environment. I&#8217;ve also created local DNS entries in the Tyrell network for the &#8216;chc.mycloud.local&#8217; and &#8216;vcde.mycloud.local&#8217; DNS names which resolve to the public IP addresses for the MyCloud vCloud Director instance and the provider CX endpoint respectively. Obviously in the real world these would be registered Internet DNS names.

In the Tyrell vCenter server when we select the &#8216;vCloud Director Extender&#8217; icon we are shown an initial view of the CX plugin interface:  
[<img loading="lazy" decoding="async" class="aligncenter wp-image-356 size-large" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCloud-Extender-UI-in-vCenter-800x622.png" alt="" width="800" height="622" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCloud-Extender-UI-in-vCenter-800x622.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCloud-Extender-UI-in-vCenter-300x233.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCloud-Extender-UI-in-vCenter-768x597.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCloud-Extender-UI-in-vCenter-193x150.png 193w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCloud-Extender-UI-in-vCenter-150x117.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCloud-Extender-UI-in-vCenter.png 1427w" sizes="(max-width: 800px) 100vw, 800px" />][1]  
Selecting the &#8216;New Provider Cloud&#8217; button opens a wizard to configure the connection to the Cloud Provider endpoints:  
[<img loading="lazy" decoding="async" class="aligncenter wp-image-357 size-large" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/02-connect-to-provider-cloud-800x411.png" alt="" width="800" height="411" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/02-connect-to-provider-cloud-800x411.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-connect-to-provider-cloud-300x154.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-connect-to-provider-cloud-768x394.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-connect-to-provider-cloud-250x128.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-connect-to-provider-cloud-150x77.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/02-connect-to-provider-cloud.png 865w" sizes="(max-width: 800px) 100vw, 800px" />][2]  
The &#8216;Provider Cloud URL&#8217; needs to be set to include the appropriate path for the vCloud Director Organisation which is being connected to (the /cloud/org/Tyrell part in this example). The user details hold the Organization Administrator role within this cloud organisation.

When clicking &#8216;Add&#8217; you will be presented with a certificate warning if the cloud provider is not using trusted/signed certificates, you can optionally select to trust these certificates if this is the case (very handy for a lab environment).

You can use the &#8216;Test&#8217; button to confirm the settings are valid &#8211; you will see a status update at the bottom of the dialog showing the status of this test:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-359" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/04-tested-ok-800x434.png" alt="" width="800" height="434" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/04-tested-ok-800x434.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-tested-ok-300x163.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-tested-ok-768x417.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-tested-ok-250x136.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-tested-ok-150x81.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/04-tested-ok.png 866w" sizes="(max-width: 800px) 100vw, 800px" />][3]

Note that even if the &#8216;Test&#8217; succeeds, there are still some circumstances to do with network connectivity that can result in the enablement process failing &#8211; this is shown in the following capture from the &#8216;Provider Clouds&#8217; tab where you can see the &#8216;Status&#8217; shows &#8216;Enable Failed&#8217;:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-360" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/05-enable-failed-800x365.png" alt="" width="800" height="365" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/05-enable-failed-800x365.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-enable-failed-300x137.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-enable-failed-768x350.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-enable-failed-250x114.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-enable-failed-150x68.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/05-enable-failed.png 1109w" sizes="(max-width: 800px) 100vw, 800px" />][4]

This is usually caused by incorrect firewall rules, NAT rules or Public Endpoint URL&#8217;s set incorrectly when the CX appliances are deployed, I&#8217;m intending to cover this in a future &#8216;Troubleshooting&#8217; part to this series of posts.

Once the networking and URLs are configured correctly you will see the new provider cloud registered under the &#8216;Provider Clouds&#8217; tab with a status of &#8216;Running&#8217;, you will also see any accessible virtual datacenters (vDC) to which you have access:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-361" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/06-all-ok-800x333.png" alt="" width="800" height="333" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/06-all-ok-800x333.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-all-ok-300x125.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-all-ok-768x319.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-all-ok-250x104.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-all-ok-150x62.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/06-all-ok.png 1176w" sizes="(max-width: 800px) 100vw, 800px" />][5]

Now that our provider cloud is properly registered, we can submit a migration request using the &#8216;Migrations&#8217; tab in the CX interface, first we will be asked if we wish to perform a &#8216;Cold&#8217; or &#8216;Warm&#8217; migration &#8211; the differences between these are well explained in the dialog. Note that &#8216;Warm&#8217; migration is not a vMotion, but does involve a period of network disconnection as the VM is cutover to the Cloud Provider. For this example we&#8217;ll select a &#8216;Warm&#8217; migration:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-362" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/07-migration-01-800x292.png" alt="" width="800" height="292" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/07-migration-01-800x292.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-migration-01-300x109.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-migration-01-768x280.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-migration-01-250x91.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-migration-01-150x55.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/07-migration-01.png 1181w" sizes="(max-width: 800px) 100vw, 800px" />][6]

Clicking &#8216;Next&#8217; takes us to an inventory view where we can select the source VM(s) to be migrated. The grey panel below the &#8216;Inventory Browser&#8217; dynamically expands to show candidate VMs from the vCenter environment. When a VM is selected the status and disk sizes are update in the right-side panel. For this example we&#8217;ve selected the &#8216;deckard&#8217; VM:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-363" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/08-migration-02-vm-select-800x641.png" alt="" width="800" height="641" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/08-migration-02-vm-select-800x641.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-migration-02-vm-select-300x241.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-migration-02-vm-select-768x616.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-migration-02-vm-select-187x150.png 187w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-migration-02-vm-select-150x120.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/08-migration-02-vm-select.png 1181w" sizes="(max-width: 800px) 100vw, 800px" />][7]

Clicking &#8216;Next&#8217; takes us on to the Target selection &#8211; here we can select the Cloud Provider, vDC, VM storage profile for the remote copy and the network to be connected to the VM in the Cloud Provider. Note that we are not L2-extending our on-premises network in this example and relying on our Cloud Provider (MyCloud) having already defined an Org vDC network for us (in this case called &#8216;Tyrell Servers&#8217;). All of the values are populated automatically from the vCloud Director environment and drop-downs allow easy select of other options. Finally we have the option when migrating multiple VMs together to group these into a single vApp rather than creating a new vApp for each VM:

[<img loading="lazy" decoding="async" class="aligncenter wp-image-364" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/09-migration-03-destination-800x648.png" alt="" width="800" height="334" />][8]

In the final migration configuration step we can specify when the VM synchronisation should start, what our target Recovery Point Objective (RPO) is in minutes and whether to provision the destination disks as &#8216;Thin&#8217; provisioned or &#8216;Thick&#8217; provisioned. Finally we can add an optional tag to reference against this job later:[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-365" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/10-migration-replication-800x639.png" alt="" width="800" height="639" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/10-migration-replication-800x639.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-migration-replication-300x240.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-migration-replication-768x614.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-migration-replication-188x150.png 188w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-migration-replication-150x120.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/10-migration-replication.png 1180w" sizes="(max-width: 800px) 100vw, 800px" />][9]

If everything has worked, you&#8217;ll now see a progress indicator against the VM in the Migrations tab. Initially the status will be &#8216;Created&#8217;:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-368" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/11-migration-progress-01-800x226.png" alt="" width="800" height="226" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/11-migration-progress-01-800x226.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-migration-progress-01-300x85.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-migration-progress-01-768x217.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-migration-progress-01-250x71.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-migration-progress-01-150x42.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/11-migration-progress-01.png 1400w" sizes="(max-width: 800px) 100vw, 800px" />][10]

Once data synchronisation begins this status will be updated to show the synchronised percentage for the migration. If you get an &#8216;Error&#8217; prior to the sync percentage moving from 0% this is almost certainly a network configuration issue (and one which I encountered frequently when first building my lab environment). I&#8217;ll cover the common causes and remedies for this more in my vCloud Extender Troubleshooting post.

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-369" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-02-800x230.png" alt="" width="800" height="230" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-02-800x230.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-02-300x86.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-02-768x221.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-02-250x72.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-02-150x43.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-02.png 1400w" sizes="(max-width: 800px) 100vw, 800px" />][11]

Once the initial synchronisation process has completed you will see the VM listed as &#8216;Cutover ready&#8217; which means it&#8217;s staged and ready to be migrated:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-370" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-03-800x233.png" alt="" width="800" height="233" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-03-800x233.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-03-300x87.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-03-768x223.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-03-250x73.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-03-150x44.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-03.png 1379w" sizes="(max-width: 800px) 100vw, 800px" />][12]

Logging in to the Tyrell vCloud Director portal at this point shows that nothing actually has been provisioned into the Tyrell VDC:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-371" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/14-migration-vcd-view-before-cutover-800x372.png" alt="" width="800" height="372" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/14-migration-vcd-view-before-cutover-800x372.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-migration-vcd-view-before-cutover-300x139.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-migration-vcd-view-before-cutover-768x357.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-migration-vcd-view-before-cutover-250x116.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-migration-vcd-view-before-cutover-150x70.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/14-migration-vcd-view-before-cutover.png 1143w" sizes="(max-width: 800px) 100vw, 800px" />][13]

Looking at the &#8216;Home&#8217; page for the CX environment in vCenter shows our VM as in a &#8216;Transition&#8217; state:

[<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-372" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/15-migration-transition.png" alt="" width="730" height="579" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/15-migration-transition.png 730w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-migration-transition-300x238.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-migration-transition-189x150.png 189w, https://kiwicloud.ninja/wp-content/uploads/2017/10/15-migration-transition-150x119.png 150w" sizes="(max-width: 730px) 100vw, 730px" />][14]

In the Migrations tab we can now select the &#8216;Start Cutover&#8217; button to actually cutover the VM to the Cloud Provider environment which opens the Cutover dialog:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-373" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/16-migration-cutover-800x356.png" alt="" width="800" height="356" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/16-migration-cutover-800x356.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-migration-cutover-300x133.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-migration-cutover-768x342.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-migration-cutover-250x111.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-migration-cutover-150x67.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/16-migration-cutover.png 1419w" sizes="(max-width: 800px) 100vw, 800px" />][15]

Clicking &#8216;Start&#8217; asks for confirmation and then performs the actual cutover to running the VM in the Cloud Provider datacenter, progress is updated during the cutover procedure:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-375" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/18-migration-cutover-progress-02-800x227.png" alt="" width="800" height="227" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/18-migration-cutover-progress-02-800x227.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-migration-cutover-progress-02-300x85.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-migration-cutover-progress-02-768x218.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-migration-cutover-progress-02-250x71.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-migration-cutover-progress-02-150x43.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/18-migration-cutover-progress-02.png 1410w" sizes="(max-width: 800px) 100vw, 800px" />][16]

When the cutover process is complete you will see the Status update:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-376" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/19-migration-cutover-done-800x214.png" alt="" width="800" height="214" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/19-migration-cutover-done-800x214.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-migration-cutover-done-300x80.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-migration-cutover-done-768x205.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-migration-cutover-done-250x67.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-migration-cutover-done-150x40.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/19-migration-cutover-done.png 1410w" sizes="(max-width: 800px) 100vw, 800px" />][17]

Looking in vCenter at this point shows the original VM still in place, but now powered off, you should probably take steps to ensure that this VM cannot be accidentally started at this point or risk having two running instances of the same VM (potentially on the same network if your network is extended to the Cloud Provider):

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-377" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/20-migration-original-powered-off-800x336.png" alt="" width="800" height="336" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/20-migration-original-powered-off-800x336.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-migration-original-powered-off-300x126.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-migration-original-powered-off-768x322.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-migration-original-powered-off-250x105.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-migration-original-powered-off-150x63.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/20-migration-original-powered-off.png 1042w" sizes="(max-width: 800px) 100vw, 800px" />][18]

Refreshing the Tyrell vCloud Director portal shows the migrated VM now running in the Tyrell Cloud Provider VDC:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-378" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/21-migration-running-in-vCloud-800x490.png" alt="" width="800" height="490" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/21-migration-running-in-vCloud-800x490.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-migration-running-in-vCloud-300x184.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-migration-running-in-vCloud-768x471.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-migration-running-in-vCloud-245x150.png 245w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-migration-running-in-vCloud-150x92.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/21-migration-running-in-vCloud.png 1020w" sizes="(max-width: 800px) 100vw, 800px" />][19]

The status in the vCloud Extender vCenter plugin also now shows the completed migration total:

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-380" src="https://kiwicloud.ninja/wp-content/uploads/2017/10/22-migration-health-view-800x420.png" alt="" width="800" height="420" srcset="https://kiwicloud.ninja/wp-content/uploads/2017/10/22-migration-health-view-800x420.png 800w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-migration-health-view-300x157.png 300w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-migration-health-view-768x403.png 768w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-migration-health-view-250x131.png 250w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-migration-health-view-150x79.png 150w, https://kiwicloud.ninja/wp-content/uploads/2017/10/22-migration-health-view.png 1035w" sizes="(max-width: 800px) 100vw, 800px" />][20]

In the next part of this series of articles I look at the options to extend L2 networking directly from a customer site into vCloud Director using CX and the changes this introduces into the migration workflow.

[Link back to Part 3][21] || [Link to Part 5][22]

As always, corrections, comments and feedback are always appreciated.

Jon.

 [1]: https://kiwicloud.ninja/wp-content/uploads/2017/10/01-vCloud-Extender-UI-in-vCenter.png
 [2]: https://kiwicloud.ninja/wp-content/uploads/2017/10/02-connect-to-provider-cloud.png
 [3]: https://kiwicloud.ninja/wp-content/uploads/2017/10/04-tested-ok.png
 [4]: https://kiwicloud.ninja/wp-content/uploads/2017/10/05-enable-failed.png
 [5]: https://kiwicloud.ninja/wp-content/uploads/2017/10/06-all-ok.png
 [6]: https://kiwicloud.ninja/wp-content/uploads/2017/10/07-migration-01.png
 [7]: https://kiwicloud.ninja/wp-content/uploads/2017/10/08-migration-02-vm-select.png
 [8]: https://kiwicloud.ninja/wp-content/uploads/2017/10/09-migration-03-destination.png
 [9]: https://kiwicloud.ninja/wp-content/uploads/2017/10/10-migration-replication.png
 [10]: https://kiwicloud.ninja/wp-content/uploads/2017/10/11-migration-progress-01.png
 [11]: https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-02.png
 [12]: https://kiwicloud.ninja/wp-content/uploads/2017/10/12-migration-progress-03.png
 [13]: https://kiwicloud.ninja/wp-content/uploads/2017/10/14-migration-vcd-view-before-cutover.png
 [14]: https://kiwicloud.ninja/wp-content/uploads/2017/10/15-migration-transition.png
 [15]: https://kiwicloud.ninja/wp-content/uploads/2017/10/16-migration-cutover.png
 [16]: https://kiwicloud.ninja/wp-content/uploads/2017/10/18-migration-cutover-progress-02.png
 [17]: https://kiwicloud.ninja/wp-content/uploads/2017/10/19-migration-cutover-done.png
 [18]: https://kiwicloud.ninja/wp-content/uploads/2017/10/20-migration-original-powered-off.png
 [19]: https://kiwicloud.ninja/wp-content/uploads/2017/10/21-migration-running-in-vCloud.png
 [20]: https://kiwicloud.ninja/wp-content/uploads/2017/10/22-migration-health-view.png
 [21]: http://152.67.105.113/2017/10/vcloud-director-extender-part-3-tenant-setup/
 [22]: http://152.67.105.113/2017/11/vcloud-director-extender-part-5-stretch-networking-l2vpn/