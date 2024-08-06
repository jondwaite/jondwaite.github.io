---
title: 'vCloud Availability 3.0 – Site Pairing & vCAv Policies'
author: Jon Waite
type: post
date: 2019-04-22T09:53:41+00:00
url: /2019/04/vcloud-availability-3-0-site-pairing-vcav-policies/
categories:
  - vCloud Availability
  - vCloud Director
  - VMware

---
The first 2 parts of this series covered the overall vCloud Availability (vCAv) architecture and the deployment and configuration of the vCAv appliances into a Cloud Provider site. Before continuing pairing sites and configuring VM replication policies, first check that all services are online and showing as healthy.

The easiest place to do this is the &#8216;System Monitoring&#8217; screen in the vApp Replication Manager portal (in my lab this is https://10.207.0.44/ui/admin for the Auckland site and https://10.200.0.44/ui/admin for the Christchurch site). The resulting panels look like this (the &#8216;Local replicators&#8217; tab has been expanded in both sites):

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-status-prior-to-pairing.png" alt="" class="wp-image-1024" width="1144" height="652" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-status-prior-to-pairing.png 1525w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-status-prior-to-pairing-300x171.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-status-prior-to-pairing-768x438.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-status-prior-to-pairing-800x456.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-status-prior-to-pairing-150x85.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-status-prior-to-pairing-250x142.png 250w" sizes="(max-width: 1144px) 100vw, 1144px" /><figcaption>System Monitoring screens for both sites prior to site pairing</figcaption></figure>
</div>

**Note:** If you have changed the SSL certificate for the vApp Replication Manager portal as mentioned at the end of my previous post, you may see the &#8216;Tunnel connectivity&#8217; item showing red with a &#8216;requires authentication&#8217; error. If so, simply access the configuration tab, click &#8216;Edit&#8217; next to the &#8216;Tunnel address&#8217; and provide the appliance password when prompted as shown in the screen below:

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" decoding="async" width="800" height="686" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-fixing-tunnel-authentication-800x686.png" alt="" class="wp-image-1025" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-fixing-tunnel-authentication-800x686.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-fixing-tunnel-authentication-300x257.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-fixing-tunnel-authentication-768x658.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-fixing-tunnel-authentication-150x129.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-fixing-tunnel-authentication-175x150.png 175w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-fixing-tunnel-authentication.png 1050w" sizes="(max-width: 800px) 100vw, 800px" /><figcaption>Re-authenticate Tunnel Service after SSL certificate change</figcaption></figure>
</div>

**Note:** I had a number of instances in my lab setup where the &#8216;Network&#8217; entry (arrowed & boxed in green above) for the vApp Replication Manager had changed to be the public URL for vCAv. If this occurs you will not be able to pair sites as the tunnel appliance will redirect the &#8216;management&#8217; traffic back to itself. To fix this, edit the entry and point this back to the internal name/IP of the vApp Replication Manager appliance and re-enter the appliance password.

Based on my experiences in testing, I strongly suggest at this stage that you do not continue attempting to pair vCAv cloud sites until you have resolved any issues and have all System Monitoring links showing as Green/Ok. I had a number of issues in my early lab attempts to configure vCAv which would likely have been avoided if I&#8217;d done this&#8230;

You also at this point need to make sure that your public API endpoint link is added to your firewall and NAT configuration so that internet access to your vCAv public API address is passed to port 8048 on the Tunnel appliance. Once configured properly, accessing the public vCAv API address from a browser should show the vCAv portal login screen:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAv-portal-login-800x436.png" alt="" class="wp-image-1030" width="600" height="327" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAv-portal-login-800x436.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAv-portal-login-300x164.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAv-portal-login-768x419.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAv-portal-login-150x82.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAv-portal-login-250x136.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/vCAv-portal-login.png 1151w" sizes="(max-width: 600px) 100vw, 600px" /><figcaption>Checking that your public vCAv URI is accessible</figcaption></figure>
</div>

### Pairing vCloud Availability Cloud Provider Sites {.wp-block-heading}

To pair the vCAv sites, first logout of any vCAv portals and go to the user login at https:<IP-address-of-vApp-Replication-Manager>/ui/login (as opposed to /ui/admin). The username will show &#8216;user@org&#8217; instead of the &#8216;root&#8217; Appliance login presented from the /admin/ui portal. Login with your vCloud Director provider credentials (e.g. &#8216;administrator@system&#8217;).

Once logged in, selecting the &#8216;Sites&#8217; option should show a screen similar to the following:

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" decoding="async" width="800" height="383" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-prior-to-pairing-800x383.png" alt="" class="wp-image-1031" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-prior-to-pairing-800x383.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-prior-to-pairing-300x144.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-prior-to-pairing-768x367.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-prior-to-pairing-150x72.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-prior-to-pairing-250x120.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-prior-to-pairing.png 1229w" sizes="(max-width: 800px) 100vw, 800px" /><figcaption>Site Pairing &#8211; Check that the endpoint URL matches your vCAv public URL</figcaption></figure>
</div>

Click &#8216;New Pairing&#8217; and provide the details for the 2nd vCAv site (Since I&#8217;m configuring the site pair on my Auckland site I enter the details for the Christchurch site to pair):

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-pairing-800x700.png" alt="" class="wp-image-1032" width="600" height="525" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-pairing-800x700.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-pairing-300x262.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-pairing-768x672.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-pairing-150x131.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-pairing-172x150.png 172w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-site-pairing.png 868w" sizes="(max-width: 600px) 100vw, 600px" /><figcaption>Use the vCAv public URL from the partner site</figcaption></figure>
</div>

Accept the SSL certificate from the paired site and you should be able to successfully pair the sites, the Sites window should now look like this:

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" decoding="async" width="800" height="179" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-paired-sites-800x179.png" alt="" class="wp-image-1033" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-paired-sites-800x179.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-paired-sites-300x67.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-paired-sites-768x172.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-paired-sites-150x34.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-paired-sites-250x56.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-paired-sites.png 929w" sizes="(max-width: 800px) 100vw, 800px" /><figcaption>Site Pairing Completed</figcaption></figure>
</div>

**Note:** It is only necessary to perform this pairing on one site providing you specify the remote appliance credentials, vCAv will automatically associate the sites in both directions.

If we select the 2nd (C00-Christchurch) site, a login button appears allowing us to authenticate as our vCloud Director provider admin user (administrator@system) to the 2nd site. After successful authentication the Sites tab will show us with a management session to both sites:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-paired-and-authenticated-800x179.png" alt="" class="wp-image-1034" width="800" height="179" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-paired-and-authenticated-800x179.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-paired-and-authenticated-300x67.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-paired-and-authenticated-768x172.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-paired-and-authenticated-150x34.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-paired-and-authenticated-250x56.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-sites-paired-and-authenticated.png 929w" sizes="(max-width: 800px) 100vw, 800px" /></figure>
</div>

If you now check the &#8216;System Monitoring&#8217; tab, you should see that both the local (to the site you are logged in to) and remote vCAv Replicators are shown and connected:

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" decoding="async" width="800" height="760" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-system-monitoring-after-pairing-800x760.png" alt="" class="wp-image-1036" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-system-monitoring-after-pairing-800x760.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-system-monitoring-after-pairing-300x285.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-system-monitoring-after-pairing-768x730.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-system-monitoring-after-pairing-150x143.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-system-monitoring-after-pairing-158x150.png 158w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-system-monitoring-after-pairing.png 1229w" sizes="(max-width: 800px) 100vw, 800px" /></figure>
</div>

Note: The odd &#8216;Address&#8217; shown for the Remote replicator (boxed in red above) is correct &#8211; this is an internal address used to reach the remote replicator appliance via the vCAv Tunnel.

### vCloud Availability Policies {.wp-block-heading}

Now that we have 2 paired vCAv sites, we can configure policies and assign these to vCloud Director tenants to allow these tenants to configure protection for their VMs. 

Again working in the vApp Replication Manager portal (https://10.207.0.44/ui/login in my lab environment) and signed in as our vCloud Director provider account we can see the vCAv policies under the &#8216;Policies&#8217; tab. The default vCAv policy assigned to all vCloud tenants by default forbids any replications:

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" decoding="async" width="800" height="468" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-01-1-800x468.png" alt="" class="wp-image-1039" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-01-1-800x468.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-01-1-300x176.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-01-1-768x449.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-01-1-150x88.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-01-1-250x146.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-01-1.png 1229w" sizes="(max-width: 800px) 100vw, 800px" /><figcaption>Creating a new vCAv Policy</figcaption></figure>
</div>

Selecting the &#8216;New&#8217; button allows us to configure a new policy:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-02.png" alt="" class="wp-image-1040" width="435" height="387" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-02.png 580w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-02-300x267.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-02-150x133.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-02-169x150.png 169w" sizes="(max-width: 435px) 100vw, 435px" /><figcaption>Configuring a new vCAv Policy</figcaption></figure>
</div>

Replication can be allowed in either direction and the maximum number of retained instances (snapshots) per VM replication can also be set between 1 and 24. The minimum allowed RPO (set to 4 hours in this example, but configurable from 5 minutes to 24 hours) prevents users of the policy configuring smaller RPOs than defined in the policy.

**Note:** You will need to configure and assign a policy in both sites (Auckland and Christchurch in my lab environment) to permit configuration of VM replication. Policies do not automatically replicate between cloud sites, neither do assignments of policies to organizations/tenants.

With the policy just created selected, we can now select the &#8216;assign&#8217; link to add tenant organizations to the policy as shown below:

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" decoding="async" width="800" height="673" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-03-800x673.png" alt="" class="wp-image-1044" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-03-800x673.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-03-300x252.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-03-768x646.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-03-150x126.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-03-178x150.png 178w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-03.png 932w" sizes="(max-width: 800px) 100vw, 800px" /><figcaption>Asigning vCAv policy to Organization</figcaption></figure>
</div>

The popup dialog that appears on clicking &#8216;Assign&#8217; allows one (or many) vCloud tenant Organizations to be associated with the specified policy.

Note: A vCloud Organization can only ever be assigned to one vCAv policy at a time, if a tenant is assigned to a new policy any previous assignments from that tenant will be removed. In the screen below I assign the new policy to the &#8216;Tyrell&#8217; tenant organization in vCloud Director:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-04.png" alt="" class="wp-image-1045" width="435" height="365" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-04.png 580w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-04-300x252.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-04-150x126.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/04/03-policies-04-179x150.png 179w" sizes="(max-width: 435px) 100vw, 435px" /><figcaption>Asigning vCAv Policy to a vCloud Tenant Organization</figcaption></figure>
</div>

In order for the tenant to be able to configure VM replication, a policy must be defined and assigned in both sites to the organization (in order to have resources at both sites the tenant must also be defined and have virtual datacenter resources assigned to them in both sites). In my lab setup the &#8216;Tyrell&#8217; organization has VDCs assigned in both sites and I have also configured an identical vCAv policy in the 2nd site and assigned it.

That is all that is required in order for tenants to be able to configure VM replication in their vCloud Director portal and perform migration and VM failover between sites.

In the next post in this series I will detail the configuration steps to link an on-premise vSphere environment to connect to a vCloud Service Provider site using vCAv.