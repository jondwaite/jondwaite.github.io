---
title: Automated Cassandra Metrics Cluster for VCD
author: Jon Waite
type: post
date: 2023-09-16T23:02:00+00:00
url: /2023/09/automated-cassandra-metrics-cluster-for-vcd/
featured_image: https://kiwicloud.ninja/wp-content/uploads/2023/09/image-1-150x97.png
categories:
  - Cassandra
  - vCloud Director
tags:
  - Cassandra
  - PowerCLI
  - PowerShell
  - vCloud Director

---
<a rel="noreferrer noopener" href="https://docs.vmware.com/en/VMware-Cloud-Director/index.html" data-type="link" data-id="https://docs.vmware.com/en/VMware-Cloud-Director/index.html" target="_blank">VMware Cloud Director (VCD)</a> has the ability to use an <a rel="noreferrer noopener" href="https://cassandra.apache.org/_/index.html" target="_blank">Apache Cassandra</a> database to store metrics on VM performance and then make these metrics available so that tenants can view the historic performance of their VMs:

<div class="wp-block-image">
  <figure class="aligncenter size-large"><a href="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-1.png"><img loading="lazy" decoding="async" width="800" height="518" src="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-1-800x518.png" alt="" class="wp-image-69133" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-1-800x518.png 800w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-1-300x194.png 300w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-1-768x498.png 768w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-1-150x97.png 150w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-1-231x150.png 231w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-1.png 1318w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>



Installing a Cassandra cluster for use by VCD is reasonably straightforward and requires a minimum of 4 servers, of which 2 are configured as &#8216;seed&#8217; nodes. There are several guides available online on how to do this, but many of these guides available don&#8217;t cover configuring the cluster with encryption between the nodes themselves or between clients (VCD) and the servers. In addition, since the 4.1.x release of Cassandra (which is now supported by the latest versions of VCD) &#8216;standard&#8217; PEM certificates are supported unlike previous versions which required the use of Java JKS keystores which simplifies the configuration steps required.

As I find myself reasonably frequently deploying VCD in lab environments, I decided this would be an interesting challenge to fully automate the build and configuration of a Cassandra cluster with SSL enabled suitable for use with VCD. My main goals were:

<ul class="wp-block-list">
  <li>
    Fully automated Cassandra cluster build from &#8216;scratch&#8217; using Powershell and PowerCLI
  </li>
  <li>
    Optionally configure SSL encryption between cluster nodes and between clients and the cluster
  </li>
  <li>
    Use standard (cloud-init) way of customising the cluster nodes
  </li>
  <li>
    Be able to use user/password and/or SSH certificate authentication to login to the cluster nodes
  </li>
  <li>
    Allow customisation of all relevant cluster node parameters (node networking, sizing, storage etc.)
  </li>
  <li>
    Allow use of an existing/external Certificate Authority (CA) to sign the SSL certificates
  </li>
  <li>
    Publish the resulting scripts so that others could use this in their own labs/environments
  </li>
</ul>

Since this post is quite long, I&#8217;ve provided links to each part of the process below. If you just want to use the resulting scripts to build a cluster I&#8217;ve published a <a rel="noreferrer noopener" href="https://github.com/jondwaite/vcd-cassandra/" target="_blank">github repo</a> which is documented with the steps required to deploy a cluster. I&#8217;ve provided two versions of the deploy script in this repo &#8211; one to configure SSL and one which allows deployment without SSL for testing.

<div class="wp-block-ht-block-toc is-style-rounded htoc htoc--position-wide toc-list-style-numbered" data-htoc-state="expanded">
  <span class="htoc__title"><span class="ht_toc_title">Table of Contents</span></span>
  
  <div class="htoc__itemswrap">
    <ol class="ht_toc_list">
      <li class="">
        <a href="#htoc-components-used-amp-environment">Components Used & Environment</a>
      </li>
      <li class="">
        <a href="#htoc-configuration-files">Configuration Files</a>
      </li>
      <li class="">
        <a href="#htoc-generate-certificates-for-the-nodes">Generate Certificates for the Nodes</a>
      </li>
      <li class="">
        <a href="#htoc-deploy-the-cluster">Deploy the Cluster</a>
      </li>
      <li class="">
        <a href="#htoc-configure-vcd-to-use-the-cluster">Configure VCD to use the Cluster</a>
      </li>
      <li class="">
        <a href="#htoc-worked-example">Final Thoughts / Conclusion</a>
      </li>
    </ol>
  </div>
</div>



### Components Used & Environment {#htoc-components-used-amp-environment.wp-block-heading}

The specific environment used to develop and test the scripts is my homelab which uses the following components and versions &#8211; other versions/combinations may work, but have not been tested:

<ul class="wp-block-list">
  <li>
    A VMware Cloud Director 10.5 environment
  </li>
  <li>
    VMware vSphere 8.01 (vCenter & Hosts)
  </li>
  <li>
    PowerShell Core v7.3.6 and VMware PowerCLI v13.1
  </li>
  <li>
    An Openssl client (only required for the SSL script version), the one included in the Git desktop client for Windows systems is suitable and works fine
  </li>
  <li>
    An Ubuntu 22.04 LTS OVA <a rel="noreferrer noopener" href="https://cloud-images.ubuntu.com/jammy/current/" target="_blank">Cloud Image</a> for the Node OS
  </li>
  <li>
    A Certificate Authority (CA) to sign the node SSL certificates (if using the SSL version of the deploy script)
  </li>
  <li>
    The scripts from <a rel="noreferrer noopener" href="https://github.com/jondwaite/vcd-cassandra" target="_blank">https://github.com/jondwaite/vcd-cassandra</a> which can be downloaded via:<br /><code data-enlighter-language="yaml" class="EnlighterJSRAW">git clone https://github.com/jondwaite/vcd-cassandra</code> (or via the download link from github)
  </li>
</ul>

The following files are included in the github repo:<figure class="wp-block-table">

<table>
  <tr>
    <th class="has-text-align-left" data-align="left">
      File
    </th>
    
    <th class="has-text-align-left" data-align="left">
      Purpose
    </th>
  </tr>
  
  <tr>
    <td class="has-text-align-left" data-align="left">
      gencsrs.ps1
    </td>
    
    <td class="has-text-align-left" data-align="left">
      Uses openssl to create private key files for each node and then generates Certificate Signing Request (CSR) files for each node in the &#8216;certs&#8217; folder. Uses cluster.ps1 to obtain node parameters and openssl.cnf for openssl configuration
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-left" data-align="left">
      openssl.cnf
    </td>
    
    <td class="has-text-align-left" data-align="left">
      Certificate configuration for the CSRs generated by gencsrs.ps1
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-left" data-align="left">
      cluster.ps1
    </td>
    
    <td class="has-text-align-left" data-align="left">
      Defines the vCenter and Cassandra node cluster parameters, used by both gencsrs.ps1 and the deploy.ps1/deploy-nossl.ps1 scripts
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-left" data-align="left">
      deploy.ps1
    </td>
    
    <td class="has-text-align-left" data-align="left">
      Build a Cassandra cluster using signed SSL certificates using the parameters from cluster.ps1
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-left" data-align="left">
      deploy-nossl.ps1
    </td>
    
    <td class="has-text-align-left" data-align="left">
      Build the Cassandra cluster using the parameters from cluster.ps1 without using SSL certificates (does not require gencsrs.ps1 to have been run or certificates to have been generated or signed)
    </td>
  </tr>
</table></figure> 

### Configuration Files {#htoc-configuration-files.wp-block-heading}

I&#8217;ve attempting to commend scripts provided throughout so hopefully the settings will be mainly obvious to anyone reading through.

**cluster.ps1**

The bulk of the cluster configuration is performed in the cluster.ps1 file, this is then used by the other scripts to define the VM deployment target and the cassandra cluster configuration. The fields required to be populated in this file should be reasonably clear.

Note that the <code data-enlighter-language="dockerfile" class="EnlighterJSRAW">$CassNodes</code> hash which defines the names & IP addresses for the cassandra nodes is critical and the <code data-enlighter-language="generic" class="EnlighterJSRAW">$CassSeeds</code> list must reference IP addresses from <code data-enlighter-language="generic" class="EnlighterJSRAW">$CassNodes</code> to determine which nodes are created as &#8216;seeds&#8217;. VCD requires a minimum cluster of 4 nodes with at least 2 of these being created as &#8216;seed&#8217; nodes. There is no checking in the script to ensure these conditions are met so pay particular attention to these two variables.

The network specified and IP addressing defined for the nodes must also be accessible to the VCD cell servers in order for VCD to be able to access the nodes in the created cassandra cluster.

**deploy.ps1 / deploy-nossl.ps1**

These are 2 versions of the same deployment script, the <code data-enlighter-language="generic" class="EnlighterJSRAW">deploy-nossl.ps1</code> version ignores certificates altogether and deploys a cassandra cluster which doesn&#8217;t use SSL between client and nodes or between the nodes. Change whichever one you intend to use, the other can be ignored.

The <code data-enlighter-language="generic" class="EnlighterJSRAW">$nodeHD</code>, <code data-enlighter-language="asm" class="EnlighterJSRAW">$nodeCPUs</code> and <code data-enlighter-language="generic" class="EnlighterJSRAW">$nodeMem</code> settings in the file determines the resources allocated to each provisioned node VM, the <code data-enlighter-language="generic" class="EnlighterJSRAW">$OVAFile</code> needs to be set to an Ubuntu 22.04-3 LTS cloud image OVA file (other releases & versions may work, but I&#8217;ve only tested against 22.04-3 LTS). This is unlikely to work with other Linux distributions, but the scripts should be (reasonably) straightforward to update to cope with other distributions.

There are some other &#8216;hard-coded&#8217; settings in these files (e.g. the timezone to be assigned to the node VMs) which can also be adjusted as necessary directly in these files. Note that the GPG signing key for installing the cassandra package is specified directly and may change/need to be updated over time &#8211; the <code data-enlighter-language="generic" class="EnlighterJSRAW">keyid:</code> provided in the file at the time of writing (Sep 2023) is correct.

**gencsrs.ps1 & openssl.cnf**

These files are only used if using the SSL deployment script (<code data-enlighter-language="generic" class="EnlighterJSRAW">deploy.ps1</code> rather than <code data-enlighter-language="generic" class="EnlighterJSRAW">deploy-nossl.ps1</code>). In <code data-enlighter-language="generic" class="EnlighterJSRAW">gencsrs.ps1</code> the path to a working openssl executable and <code data-enlighter-language="generic" class="EnlighterJSRAW">$SNRoot</code> value should be updated as appropriate to your environment as should the <code data-enlighter-language="generic" class="EnlighterJSRAW">openssl.cnf</code> entries for certificate Country/State/City etc.

### Generate Certificates for the Nodes {#htoc-generate-certificates-for-the-nodes.wp-block-heading}

In order to be able to deploy a cassandra cluster where the client-to-node and node-to-node communication is encrypted with SSL, each cassandra node needs a certificate signed by a trusted CA. In order to provide these, the <code data-enlighter-language="generic" class="EnlighterJSRAW">gencsrc.ps1</code> generates a private key file <code data-enlighter-language="generic" class="EnlighterJSRAW">&lt;nodename&gt;.key</code> and a Certificate Signing Request (CSR) file <code data-enlighter-language="generic" class="EnlighterJSRAW">&lt;nodename&gt;.csr</code> in the &#8216;certs&#8217; folder (by default). These CSRs should then be submitted to a Certficate Authority (CA) to generate the actual certificate files. The signed certificate files generated should be saved as <code data-enlighter-language="generic" class="EnlighterJSRAW">&lt;nodename&gt;.crt</code> in the same folder as the key and CSR files.

A <code data-enlighter-language="postgresql" class="EnlighterJSRAW">chain.crt</code> file also needs to be created in the same location as the other certificate files which contains the public certificate chain of the signing CA (and any subordinate CAs). The order of certificates in this file should be any intermediate/signing certificates first followed by the root CA certificate as standard base64 encoded (PEM) format. This is necessary so that the trusted chain can be added to the cassandra node configurations to automatically trust the certificates presented by each node.

### Deploy the Cluster {#htoc-deploy-the-cluster.wp-block-heading}

Once the files have been configured as described, open a PowerShell prompt and login to the target platform vCenter environment using Connect-VIServer and a user account with appropriate permissions to create the VMs.

Next run either <code data-enlighter-language="generic" class="EnlighterJSRAW">deploy.ps1</code> (or <code data-enlighter-language="diff" class="EnlighterJSRAW">deploy-nossl.ps1</code>) as required and the script should create each node VM in turn and configure it. The VM configuration steps are provided by the <code data-enlighter-language="generic" class="EnlighterJSRAW">$CloudConfig</code> string which is built during the script execution for each node. <code data-enlighter-language="generic" class="EnlighterJSRAW">$CloudConfig</code> is simply a cloud-init configuration which is then converted to Base64 and added to the user_data configuration supplied to the VM deployment by <code data-enlighter-language="generic" class="EnlighterJSRAW">Import-Vapp</code>

This performs the following configuration for each node server:

<ul class="wp-block-list">
  <li>
    Updates all packages to the latest versions (apt update & upgrade)
  </li>
  <li>
    Sets the VM hostname
  </li>
  <li>
    Sets the VM timezone
  </li>
  <li>
    Sets the VM &#8216;ubuntu&#8217; user password and sets this to not be expired (no forced password change on first login)
  </li>
  <li>
    Specifies the repository source and GPG key to install the Apache Cassandra package
  </li>
  <li>
    Installs Java &#8216;default-jdk&#8217; (required for Cassandra), the Cassandra package itself and the net-tools package
  </li>
  <li>
    Creates networking configuration files to assign the provided static IP address and networking information to the VM
  </li>
  <li>
    Creates certificate files for cassandra in the /etc/cassandra/certs folder (SSL script only)
  </li>
  <li>
    Creates a configuration file cassandra.yaml for cassandra (including the appropriate Cassandra SSL parameters in the SSL script)
  </li>
  <li>
    For the last node to be installed (only), creates a &#8216;firstboot.sh&#8217; file and set this to run on first boot which changes the cassandra user database password to that specified in <code data-enlighter-language="generic" class="EnlighterJSRAW">cluster.ps1</code> and then removes itself
  </li>
  <li>
    Creates a module blacklist entry for the floppy drive to prevent console Ubuntu errors
  </li>
  <li>
    Enables SSH password authentication (disabled by default in Ubuntu cloud images)
  </li>
  <li>
    Disables IPV6 networking
  </li>
  <li>
    Enables the Ubuntu firewall and creates rules to allow Cassandra and ssh traffic to pass
  </li>
  <li>
    Finally, restarts the node to ensure all updates and changes are applied
  </li>
</ul>

That&#8217;s quite a list, but it should be easy to identify each of these activities in the deploy.ps1 script.

The script then configures the destination VM parameters and changes the number of CPU cores, RAM and disk allocated to the VM as necessary.

Finally the VM is started and the script waits 5 minutes for the node configuration to be completed before deploying the next node. You can change this interval, but due to the large number of VM configurations made by cloud-init (and the following reboot) I&#8217;ve found this is generally a realistic value. Note that deploying all nodes without this pause can be done, but Cassandra has issues properly forming the cluster if multiple nodes attempt to join the cluster simultaneously so this pause provides a &#8216;safer&#8217; way to ensure the cluster forms successfully.

Note that the <code data-enlighter-language="generic" class="EnlighterJSRAW">firstboot.sh</code> script which runs on the last node to set the cassandra database user password also waits an additional 30 seconds prior to running to allow the cluster to &#8216;settle&#8217;, this password change only needs to be done on a single node since the &#8216;cassandra&#8217; database user exists for the entire cluster.

Once the cluster is deployed, operation can be checked using <code data-enlighter-language="generic" class="EnlighterJSRAW">nodetool status</code> to confirm that all nodes have been successfully deployed and joined to the cluster (&#8216;UN&#8217; status for each node):

<div class="wp-block-image">
  <figure class="aligncenter size-large"><a href="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-2.png"><img loading="lazy" decoding="async" width="800" height="174" src="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-2-800x174.png" alt="" class="wp-image-69148" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-2-800x174.png 800w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-2-300x65.png 300w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-2-768x168.png 768w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-2-150x33.png 150w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-2-250x55.png 250w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-2.png 894w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>

If any of the nodes have a status other than &#8216;UN&#8217; the cluster hasn&#8217;t formed correctly and consider allowing more time between the node deployments (change the Start-Sleep timer towards the end of <code data-enlighter-language="generic" class="EnlighterJSRAW">deploy.ps1</code>/<code data-enlighter-language="generic" class="EnlighterJSRAW">deploy-nossl.ps1</code>) and trying to redeploy.

### Configure VCD to use the Cluster {#htoc-configure-vcd-to-use-the-cluster.wp-block-heading}

When the deploy.ps1 (or deploy-nossl.ps1) script completes successfully it will output the command which needs to be run on the VCD cell to configure VCD to use the deployed cassandra cluster:

<div class="wp-block-image">
  <figure class="aligncenter size-full"><a href="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-3.png"><img loading="lazy" decoding="async" width="625" height="604" src="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-3.png" alt="" class="wp-image-69156" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-3.png 625w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-3-300x290.png 300w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-3-150x145.png 150w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-3-155x150.png 155w" sizes="(max-width: 625px) 100vw, 625px" /></a></figure>
</div>



As mentioned before, if you are using the <code data-enlighter-language="generic" class="EnlighterJSRAW">deploy-nossl.ps1</code> script you&#8217;ll also need to set &#8216;<code data-enlighter-language="generic" class="EnlighterJSRAW">cassandra.use.ssl=0</code>&#8216; in the VCD <code data-enlighter-language="generic" class="EnlighterJSRAW">global.properties</code> file or VCD won&#8217;t connect to the cluster.

If everything has been successful, pasting the cell-management-tool command line into VCD shoud give the following:

<div class="wp-block-image">
  <figure class="aligncenter size-large"><a href="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-4.png"><img loading="lazy" decoding="async" width="800" height="440" src="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-4-800x440.png" alt="" class="wp-image-69158" srcset="https://kiwicloud.ninja/wp-content/uploads/2023/09/image-4-800x440.png 800w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-4-300x165.png 300w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-4-768x422.png 768w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-4-150x82.png 150w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-4-250x137.png 250w, https://kiwicloud.ninja/wp-content/uploads/2023/09/image-4.png 986w" sizes="(max-width: 800px) 100vw, 800px" /></a></figure>
</div>



Once VCD has been successfully configured, each cell server will need the VCD service to be restarted (<code data-enlighter-language="generic" class="EnlighterJSRAW">service restart vmware-vcd</code>).

### Final Thoughts / Conclusion {#htoc-worked-example.wp-block-heading}

In this post I&#8217;ve detailed a method and code to deploy a functional Cassandra cluster with SSL enabled which is suitable for use as the metrics store in a VCD environment. The scripts I&#8217;ve provided in the <a rel="noreferrer noopener" href="https://github.com/jondwaite/vcd-cassandra" target="_blank">github repo</a> should also be easily understandable and adjustable to suit other use-cases too. In particular I&#8217;ve been impressed at how well a cloud-init user_data code block or script can be integrated into a deployment workflow by setting the OVF parameters available with the PowerCLI Import-vApp cmdlet.

Hopefully this post will prove useful to those needing to deploy Cassandra clusters with SSL encryption enabled, or serve as a starting point for such deployments.

One possible enhancement would be to automate the signing of the CSRs generated and to automatically place the generated certificates which I may look at in future.

As always, comments and feedback appreciated,

Jon.