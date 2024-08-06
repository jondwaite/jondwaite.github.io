---
title: Dynamic Persistent Volumes with CSE Kubernetes and Ceph
author: Jon Waite
type: post
date: 2019-01-29T02:22:54+00:00
url: /2019/01/dynamic-persistent-volumes-with-cse-kubernetes-and-ceph/
categories:
  - Container Service Extension
  - Containers
  - Kubernetes
  - vCloud Director
tags:
  - ceph
  - container-service-extension
  - CSE
  - dynamic persistent volumes
  - k8s
  - Kubernetes
  - vCloud Director

---
## Introduction {.wp-block-heading}

Application containerization with <a rel="noopener" href="https://www.docker.com/" target="_blank">Docker</a> is fast becoming the default deployment pattern for many business applications and <a rel="noopener" href="https://kubernetes.io/" target="_blank">Kubernetes (k8s)</a> the method of managing these workloads. While containers generally should be stateless and ephemeral (able to be deployed, scaled and deleted at will) almost all business applications require data persistence of some form. In some cases it is appropriate to offload this to an external system (a database, file store or object store in public cloud environments are common for example).

This doesn’t cover all storage requirements though, and if you are running k8s in your own environment or in a hosted service provider environment you may not have access to compatible or appropriate storage. One solution for this is to build a storage platform alongside a Kubernetes cluster which can provide storage persistence while operating in a similar deployment pattern to the k8s cluster itself (scalable, clustered, highly available and no single points of failure).

VMware <a href="https://vmware.github.io/container-service-extension/" target="_blank" rel="noopener">Container Service Extension (CSE)</a> for <a href="https://docs.vmware.com/en/vCloud-Director/index.html" target="_blank" rel="noopener">vCloud Director (vCD)</a> is an automated way for customers of vCloud powered service providers to easily deploy, scale and manage k8s clusters, however CSE currently only provides a limited storage option (an NFS storage server added to the cluster) and k8s persistent volumes (PVs) have to be pre-provisioned in NFS and assigned to containers/pods rather than being generated on-demand. This can also cause availability, scale and performance issues caused by the pod storage being located on a single server VM.

There is certainly no ‘right’ answer to the question of persistent storage for k8s clusters – often the choice will be driven by what is available in the platform you are deploying to and the security, availability and performance requirements for this storage.

In this post I will detail a deployment using a <a rel="noopener" href="https://ceph.com/" target="_blank" class="broken_link">ceph</a> storage cluster to provide a highly available and scalable storage platform and the configuration required to enable a CSE deployed k8s cluster to use dynamic persistent volumes (DPVs) in this environment.

Due to the large number of servers/VMs involved, and the possibility of confusion / working on the wrong server console &#8211; I&#8217;ve added buttons like this<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png" alt="" class="wp-image-881" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> prior to each section to show which system(s) the commands should be used on.

### Disclaimer {.wp-block-heading}

**I am not an expert in Kubernetes or ceph and have figured out most of the contents in this post from documentation, forums, google and (sometimes) trial and error. Refer to the documentation and support resources at the links at the end of this post if you need the ‘proper’ documentation on these components. Please do not use anything shown in this post in a production environment without appropriate due diligence and making sure you understand what you are doing (and why!).**

## Solution Overview {.wp-block-heading}

Our solution is going to be based on a minimal viable installation of ceph with a CSE cluster consisting of 4 ceph nodes (1 admin and 3 combined OSD/mon/mgr nodes) and a 4 node Kubernetes cluster (1 master and 3 worker nodes). There is no requirement for the OS in the ceph cluster and the kubernetes cluster to be the same, however it does make it easier if the packages used for ceph are at the same version which is easier to achieve using the same base OS for both clusters. Since CSE currently only has templates for Ubuntu 16.04 and PhotonOS, and due to the lack of packages for the ‘mimic’ release of ceph on PhotonOS, this example will use Ubuntu 16.04 LTS as the base OS for all servers.

The diagram below shows the components required to be deployed – in the lab environment I’m using the DNS and NTP servers already exist:<figure class="wp-block-image">

<img decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/solution-overview-network2.png" alt="solution overview" /> </figure> 

  
Note: In production ceph clusters, the monitor (mon) service should run on separate machines from the nodes providing storage (OSD nodes), but for a test/dev environment there is no issue running both services on the same nodes.

### Pre-requisites {.wp-block-heading}

You should ensure that you have the following enabled and configured in your environment before proceeding:

<table class="wp-block-table">
  <tr>
    <td>
      <strong>Configuration Item</strong>
    </td>
    
    <td>
      <strong>Requirement</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      DNS
    </td>
    
    <td>
      Have a DNS server available and add host (‘A’) records for each of the ceph servers. Alternatively it should be possible to add /etc/hosts records on each node to avoid the need to configure DNS. Note that this is only required for the ceph nodes to talk to each other, the kubernetes cluster uses direct IP addresses to contact the ceph cluster.
    </td>
  </tr>
  
  <tr>
    <td>
      NTP
    </td>
    
    <td>
      Have an available NTP time source on your network, or access to external ntp servers
    </td>
  </tr>
  
  <tr>
    <td>
      Static IP Pool
    </td>
    
    <td>
      Container Service Extension (CSE) requires a vCloud OrgVDC network with sufficient addresses available from a static IP pool for the number of kubernetes nodes being deployed
    </td>
  </tr>
  
  <tr>
    <td>
      SSH Key Pair
    </td>
    
    <td>
      Generated SSH key pair to be used to administer the deployed CSE servers. This could (optionally) also be used to administer the ceph servers
    </td>
  </tr>
  
  <tr>
    <td>
      VDC Capacity
    </td>
    
    <td>
      Ensure you have sufficient resources (Memory, CPU, Storage and number of VMs) in your vCD VDC to support the desired cluster sizes
    </td>
  </tr>
</table>

## Ceph Storage Cluster {.wp-block-heading}

The process below describes installing and configuring a ceph cluster on virtualised hardware. If you have an existing ceph cluster available or are building on physical hardware it&#8217;s best to follow the ceph official documentation at <a rel="noreferrer noopener" href="http://docs.ceph.com/docs/mimic/start/" target="_blank" class="broken_link">this link</a> for your circumstances.

### Ceph Server Builds {.wp-block-heading}

The 4 ceph servers can be built using any available hardware or virtualisation platform, in this exercise I’ve built them from an Ubuntu 16.04 LTS server template with 2 vCPUs and 4GB RAM for each in the same vCloud Director environment which will be used for deployment of the CSE kubernetes cluster. There are no special requirements for installing/configuring the base Operating System for the ceph cluster. If you are using a different Linux distribution then check the ceph documentation for the appropriate steps for your distribution.

On the 3 storage nodes (ceph01, ceph02 and ceph03) add a hard disk to the server which will act as the storage for the ceph Object Storage Daemon (OSD) – the storage pool which will eventually be useable in Kubernetes. In this example I’ve added a 50GB disk to each of these VMs.

Once the servers are deployed the following are performed on each server to update their repositories and upgrade any modules to current security levels. We will also upgrade the Linux kernel to a more up-to-date version by enabling the Ubuntu Hardware Extension (HWE) kernel which resolves some compatibility issues between ceph and older Linux kernel versions.<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png" alt="" class="wp-image-881" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="false" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install --install-recommends linux-generic-hwe-16.04 -y</pre>

Each server should now be restarted to ensure the new Linux kernel is loaded and any added storage disks are recognised.

### Ceph Admin Account {.wp-block-heading}

We need a user account configured on each of the ceph servers to allow ceph-deploy to work and to co-ordinate access, this account must NOT be named &#8216;ceph&#8217; due to potential conflicts in the ceph-deploy scripts, but can be called just about anything else. In this lab environment I&#8217;ve used &#8216;cephadmin&#8217;. First we create the account on each server and set the password, the 3rd line permits the cephadmin user to use &#8216;sudo&#8217; without a password which is required for the ceph-deploy script:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png" alt="" class="wp-image-881" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo useradd -d /home/cephadmin -m cephadmin -s /bin/bash
$ sudo passwd cephadmin
$ echo "cephadmin ALL = (root) NOPASSWD:ALL" > /etc/sudoers.d/cephadmin</pre>

From now on, (unless specified) use the new cephadmin login to perform each step. Next we need to generate an SSH key pair for the ceph admin user and copy this to the authorized-keys file on each of the ceph nodes.

Execute the following on the ceph admin node (as cephadmin):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ssh-keygen -t rsa</pre>

Accept the default path (/home/cephadmin/.ssh/id\_rsa) and don&#8217;t set a key passphrase. You should copy the generated .ssh/id\_rsa (private key) file to your admin workstation so you can use it to authenticate to the ceph servers.

Next, enable password logins (temporarily) on the storage nodes (ceph01,2 & 3) by running the following on each node:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-nodes-btn.png" alt="" class="wp-image-882" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo sed -i "s/.*PasswordAuthentication.*/PasswordAuthentication yes/g" /etc/ssh/sshd_config
$ sudo systemctl restart sshd</pre>

Now copy the cephadmin public key to each of the other ceph nodes by running the following (again only on the admin node):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ssh-keyscan -t rsa ceph01 >> ~/.ssh/known_hosts
$ ssh-keyscan -t rsa ceph02 >> ~/.ssh/known_hosts
$ ssh-keyscan -t rsa ceph03 >> ~/.ssh/known_hosts
$ ssh-copy-id cephadmin@ceph01
$ ssh-copy-id cephadmin@ceph02
$ ssh-copy-id cephadmin@ceph03</pre>

You should now confirm you can ssh to each storage node as the cephadmin user from the admin node without being prompted for a password:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ssh cephadmin@ceph01 sudo hostname
ceph01
$ ssh cephadmin@ceph02 sudo hostname
ceph02
$ ssh cephadmin@ceph03 sudo hostname
ceph03</pre>

If everything is working correctly then each command will return the appropriate hostname for each storage node without any password prompts.

Optional: It is now safe to re-disable password authentication on the ceph servers if required (since public key authentication will be used from now on) by:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png" alt="" class="wp-image-881" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo sed -i "s/.*PasswordAuthentication.*/PasswordAuthentication no/g" /etc/ssh/sshd_config
$ sudo systemctl restart sshd</pre>

You&#8217;ll need to resolve any authentication issues before proceeding as the ceph-deploy script relies on being able to obtain sudo-level remote access to all of the storage nodes to install ceph successfully.

You should also at this stage confirm that you have time synchronised to an external source on each ceph node so that the server clocks agree, by default on Ubuntu 16.04 timesyncd is configured automatically so nothing needs to be done here in our case. You can check this on Ubuntu 16.04 by running timedatectl:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png" alt="" class="wp-image-881" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> <figure class="wp-block-image">[<img decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image_thumb.png" alt="image" />][1]<figcaption>Checking time/date settings using timedatectl</figcaption></figure> 

For some Linux distributions you may need to create firewall rules at this stage for ceph to function, generally port 6789/tcp (for mon) and the range 6800 to 7300 tcp (for OSD communication) need to be open between the cluster nodes. The default firewall settings in Ubuntu 16.04 allow all network traffic so this is not required (however, do not use this in a production environment without configuring appropriate firewalling).

### Ceph Installation {.wp-block-heading}

On all nodes and signed-in as the cephadmin user (**important!**)  
Add the release key: <figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png" alt="" class="wp-image-881" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -</pre>

Add ceph packages to your repository:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png" alt="" class="wp-image-881" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ echo deb https://download.ceph.com/debian-mimic/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list</pre>

On the admin node only, update and install ceph-deploy:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo apt update; sudo apt install ceph-deploy -y</pre>

On all nodes, update and install ceph-common:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png" alt="" class="wp-image-881" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-both-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo apt update; sudo apt install ceph-common -y</pre>

**Note:** Installing ceph-common on the storage nodes isn&#8217;t strictly required as the ceph-deploy script can do this during cluster initiation, but pre-installing it in this way pulls in several dependencies (e.g. python v2 and associated modules) which can prevent ceph-deploy from running if not present so it is easier to do this way.

Next again working on the admin node logged in as cephadmin, make a directory to store the ceph cluster configuration files and change to that directory. Note that ceph-deploy will use and write files to the current directory so make sure you are in this folder whenever making changes to the ceph configuration.<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo apt install ceph-deploy -y</pre>

Now we can create the initial ceph cluster from the admin node, use ceph-deploy with the &#8216;new&#8217; switch and supply the monitor nodes (in our case all 3 nodes will be both monitors and OSD nodes). Make sure you do NOT use sudo for this command and only run on the admin node:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ceph-deploy new ceph01 ceph02 ceph03</pre>

If everything has run correctly you&#8217;ll see output similar to the following:<figure class="wp-block-image">

[<img decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image_thumb-1.png" alt="image" />][2]<figcaption>ceph-deploy output</figcaption></figure> 

Checking the contents of the ~/mycluster/ folder should show the cluster configuration files have been added:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ls -al ~/mycluster
total 24
drwxrwxr-x 2 cephadmin cephadmin 4096 Jan 25 01:03 .
drwxr-xr-x 5 cephadmin cephadmin 4096 Jan 25 00:57 ..
-rw-rw-r-- 1 cephadmin cephadmin  247 Jan 25 01:03 ceph.conf
-rw-rw-r-- 1 cephadmin cephadmin 7468 Jan 25 01:03 ceph-deploy-ceph.log
-rw------- 1 cephadmin cephadmin   73 Jan 25 01:03 ceph.mon.keyring</pre>

The ceph.conf file will look something like this:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ cat ~/mycluster/ceph.conf
fsid = 98ca274e-f79b-4092-898a-c12f4ed04544
mon_initial_members = ceph01, ceph02, ceph03
mon_host = 192.168.207.201,192.168.207.202,192.168.207.203
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx</pre>

Run the ceph installation for the nodes (again from the admin node only):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ceph-deploy install ceph01 ceph02 ceph03</pre>

This will run through the installation of ceph and pre-requisite packages on each node, you can check the ceph-deploy-ceph.log file after deployment for any issues or errors.

### Ceph Configuration {.wp-block-heading}

Once you&#8217;ve successfully installed ceph on each node, use the following (again from only the admin node) to deploy the initial ceph monitor services:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ceph-deploy mon create-initial</pre>

If all goes well you&#8217;ll get some messages at the completion of this process showing the keyring files being stored in your &#8216;mycluster&#8217; folder, you can check these exist:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ls -al ~/mycluster
total 168
drwxrwxr-x 2 cephadmin cephadmin   4096 Jan 25 01:17 .
drwxr-xr-x 5 cephadmin cephadmin   4096 Jan 25 00:57 ..
-rw------- 1 cephadmin cephadmin    113 Jan 25 01:17 ceph.bootstrap-mds.keyring
-rw------- 1 cephadmin cephadmin    113 Jan 25 01:17 ceph.bootstrap-mgr.keyring
-rw------- 1 cephadmin cephadmin    113 Jan 25 01:17 ceph.bootstrap-osd.keyring
-rw------- 1 cephadmin cephadmin    113 Jan 25 01:17 ceph.bootstrap-rgw.keyring
-rw------- 1 cephadmin cephadmin    151 Jan 25 01:17 ceph.client.admin.keyring
-rw-rw-r-- 1 cephadmin cephadmin    247 Jan 25 01:03 ceph.conf
-rw-rw-r-- 1 cephadmin cephadmin 128136 Jan 25 01:17 ceph-deploy-ceph.log
-rw------- 1 cephadmin cephadmin     73 Jan 25 01:03 ceph.mon.keyring</pre>

To avoid having to specify the monitor node address and ceph.client.admin.keyring path in every command, we can now deploy these to each node so they are available automatically. Again working from the &#8216;mycluster&#8217; folder on the admin node:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ceph-deploy admin cephadmin ceph01 ceph02 ceph03</pre>

This should give the following:<figure class="wp-block-image">

[<img decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image_thumb-2.png" alt="image" />][3]</figure> 

Next we need to deploy the manager (&#8216;mgr&#8217;) service to the OSD nodes, again working from the &#8216;mycluster&#8217; folder on the admin node:

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ceph-deploy mgr create ceph01 ceph02 ceph03</pre>

At this stage we can check that all of the mon and mgr services are started and ok by running (on the admin node):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo ceph -s
  cluster:
    id:     98ca274e-f79b-4092-898a-c12f4ed04544
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph01,ceph02,ceph03
    mgr: ceph01(active), standbys: ceph02, ceph03
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:</pre>

As you can see, the manager (&#8216;mgr&#8217;) service is installed on all 3 nodes but only active on the first and in standby mode on the other 2 &#8211; this is normal and correct. The monitor (&#8216;mon&#8217;) service is running on all of the storage nodes.

Next we can configure the disks attached to our storage nodes for use by ceph. Ensure that you know and use the correct identifier for your disk devices (in this case, we are using the 2nd SCSI disk attached to the storage node VMs which is at /dev/sdb so that&#8217;s what we&#8217;ll use in the commands below). As before, run the following only on the admin node:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ceph-deploy osd create --data /dev/sdb ceph01
$ ceph-deploy osd create --data /dev/sdb ceph02
$ ceph-deploy osd create --data /dev/sdb ceph03</pre>

For each command the last line of the logs shown when run should be similar to &#8216;Host ceph01 is now ready for osd use.&#8217;

We can now check the overall cluster health with:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ ssh ceph01 sudo ceph health
HEALTH_OK
$ ssh ceph01 sudo ceph -s
  cluster:
    id:     98ca274e-f79b-4092-898a-c12f4ed04544
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph01,ceph02,ceph03
    mgr: ceph01(active), standbys: ceph02, ceph03
    osd: 3 osds: 3 up, 3 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   3.0 GiB used, 147 GiB / 150 GiB avail
    pgs:</pre>

As you can see, the 3 x 50GB disks have now been added and the total (150 GiB) capacity is available under the data: section.

Now we need to create a ceph storage pool ready for Kubernetes to consume from &#8211; the default name of this pool is &#8216;rbd&#8217; (if not specified), but it is strongly recommended to name it differently from the default when using for k8s so I&#8217;ve created a storage pool called &#8216;kube&#8217; in this example (again running from the mycluster folder on the admin node):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo ceph osd pool create kube 30 30
pool 'kube' created</pre>

The two &#8217;30&#8217;s are important &#8211; you should review the ceph documentation <a href="http://docs.ceph.com/docs/mimic/rados/configuration/pool-pg-config-ref" target="_blank" rel="noopener" class="broken_link">here</a> for Pool, PG and CRUSH configuration to establish values for PG and PGP appropriate to your environment.

We now associated this pool with the rbd (RADOS block device) application so it is available to be used as a RADOS block device:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo ceph osd pool application enable kube rbd
enabled application 'rbd' on pool 'kube'</pre>

### Testing Ceph Storage {.wp-block-heading}

The easiest way to test our ceph cluster is working correctly and can provide storage is to attempt creating and using a new RADOS Block Device (rbd) volume from our admin node.

Before this will work we need to tune the rbd features map by editing ceph.conf on our client to disable rbd features that aren&#8217;t available in our Linux kernel (on admin/client node):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ echo "rbd_default_features = 7" | sudo tee -a /etc/ceph/ceph.conf
rbd_default_features = 7</pre>

Now we can test creating a volume:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo rbd create --size 1G kube/testvol01</pre>

Confirm that the volume exists:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo rbd ls kube
testvol01</pre>

Get information on our volume:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo rbd info kube/testvol01
rbd image 'testvol01':
        size 1 GiB in 256 objects
        order 22 (4 MiB objects)
        id: 10e96b8b4567
        block_name_prefix: rbd_data.10e96b8b4567
        format: 2
        features: layering, exclusive-lock
        op_features:
        flags:
        create_timestamp: Sun Jan 27 08:50:45 2019</pre>

Map the volume to our admin host (which creates the block device /dev/rbd0):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo rbd map kube/testvol01
/dev/rbd0</pre>

Now we can create a temporary mount folder, make a filesystem on our volume and mount it to our temporary mount:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="23" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo mkdir /testmnt
$ sudo mkfs.xfs /dev/rbd0
meta-data=/dev/rbd0              isize=512    agcount=9, agsize=31744 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=0
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=1024   swidth=1024 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=8 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
$ sudo mount /dev/rbd0 /testmnt
$ df -vh
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           395M  5.7M  389M   2% /run
/dev/sda1       9.6G  2.2G  7.4G  24% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/sda15      105M  3.4M  101M   4% /boot/efi
tmpfs           395M     0  395M   0% /run/user/1001
/dev/rbd0      1014M   34M  981M   4% /testmnt</pre>

We can see our volume has been mounted successfully and can now be used as any other disk.

To tidy up and remove our test volume:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo umount /dev/rbd0
$ sudo rbd unmap kube/testvol01
$ sudo rbd remove kube/testvol01
Removing image: 100% complete...done.
$ sudo rmdir /testmnt</pre>



## Kubernetes CSE Cluster {.wp-block-heading}

Using VMware Container Service Extension (CSE) makes it easy to deploy and configure a base Kubernetes cluster into our vCloud Director platform. I previously wrote a post [here][4] with a step-by-step guide to using CSE.

First we need an ssh key pair to provide to the CSE nodes as they are deployed to allow us to access them. You could re-use the cephadmin key-pair created in the previous section, or generate a new set. As I&#8217;m using Windows as my client OS I used the **puttygen** utility included in the [PuTTY][5] package to generate a new keypair and save them to a .ssh directory in my home folder.

**Important Note:** Check your public key file in a text editor prior to deploying the cluster, if it looks like this:<figure class="wp-block-image">

<img decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-12.png" alt="This image has an empty alt attribute; its file name is image-12.png" /> <figcaption>Public key as generated by PuTTYGen (incorrect)  
</figcaption></figure> 

You will need to change it to be all on one line starting &#8216;ssh-rsa&#8217; and with none of the extra text as follows:<figure class="wp-block-image">

<img decoding="async" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-11.png" alt="This image has an empty alt attribute; its file name is image-11.png" /> </figure> 

If you do not make this change this you won&#8217;t be able to authenticate to your cluster nodes once deployed.

Next we login to vCD using the vcd-cli (see my post linked above if you need to install/configure vcd-cli and the CSE extension):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png" alt="" class="wp-image-879" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> <figure class="wp-block-image"><img loading="lazy" decoding="async" width="527" height="50" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-4.png" alt="" class="wp-image-832" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-4.png 527w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-4-300x28.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-4-150x14.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-4-250x24.png 250w" sizes="(max-width: 527px) 100vw, 527px" /><figcaption>Logging in to vcd-cli</figcaption></figure> 

Now we can see what virtual Datacenters (VDCs) are available to us:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png" alt="" class="wp-image-879" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> <figure class="wp-block-image"><img loading="lazy" decoding="async" width="312" height="68" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-5.png" alt="" class="wp-image-833" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-5.png 312w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-5-300x65.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-5-150x33.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-5-250x54.png 250w" sizes="(max-width: 312px) 100vw, 312px" /><figcaption>Showing available VDCs</figcaption></figure> 

If we had multiple VDCs available, we need to select which one is &#8216;in_use&#8217; (active) for deployment of our cluster using &#8216;vcd vdc use &#8220;<VDC Name>&#8221;&#8216;. In this case we only have a single VDC and it&#8217;s already active/in use.

We can get the information of our VDC which will help us fill out the required properties when creating our k8s cluster:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png" alt="" class="wp-image-879" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> <figure class="wp-block-image"><img loading="lazy" decoding="async" width="485" height="633" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-6.png" alt="" class="wp-image-834" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-6.png 485w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-6-230x300.png 230w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-6-115x150.png 115w" sizes="(max-width: 485px) 100vw, 485px" /><figcaption>VDC Properties returned by vdc info</figcaption></figure> 

We will be using the &#8216;Tyrell Servers A03&#8217; network (where our ceph cluster exists) and the &#8216;A03 VSAN Performance&#8217; storage profile for our cluster.

To get the options available when creating a cluster we can see the cluster creation help:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png" alt="" class="wp-image-879" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> <figure class="wp-block-image"><img loading="lazy" decoding="async" width="636" height="264" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-7.png" alt="" class="wp-image-835" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-7.png 636w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-7-300x125.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-7-150x62.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-7-250x104.png 250w" sizes="(max-width: 636px) 100vw, 636px" /><figcaption>CSE cluster create options</figcaption></figure> 

Now we can go ahead and create out Kubernetes cluster with CSE:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png" alt="" class="wp-image-879" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> <figure class="wp-block-image"><img loading="lazy" decoding="async" width="1271" height="138" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-17.png" alt="" class="wp-image-853" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-17.png 1271w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-17-300x33.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-17-768x83.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-17-800x87.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-17-150x16.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-17-250x27.png 250w" sizes="(max-width: 1271px) 100vw, 1271px" /></figure> 

Looking in vCloud Director we can see the new vApp and VMs deployed:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="800" height="365" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-18-800x365.png" alt="" class="wp-image-854" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-18-800x365.png 800w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-18-300x137.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-18-768x350.png 768w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-18-150x68.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-18-250x114.png 250w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-18.png 1275w" sizes="(max-width: 800px) 100vw, 800px" /> </figure> 

We obtain the kubectl config of our cluster and store this for later use (make the .kube folder first if it doesn&#8217;t already exist):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png" alt="" class="wp-image-879" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">C:\Users\admin>vcd cse cluster config k8sceph > .kube\config</pre>

And get the details of our k8s nodes from vcd-cli:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png" alt="" class="wp-image-879" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> <figure class="wp-block-image"><img loading="lazy" decoding="async" width="336" height="122" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-19.png" alt="" class="wp-image-855" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-19.png 336w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-19-300x109.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-19-150x54.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-19-250x91.png 250w" sizes="(max-width: 336px) 100vw, 336px" /></figure> 

Next we need to update and install the ceph client on each cluster node &#8211; run the following on each node (including the master). To do this we can connect via ssh as root using the key pair we specified when creating the cluster.<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-both-btn.png" alt="" class="wp-image-884" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-both-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-both-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="false" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
OK
# echo deb https://download.ceph.com/debian-mimic/ $(lsb_release -sc) main > /etc/apt/sources.list.d/ceph.list
# apt-get update
# apt-get install --install-recommends linux-generic-hwe-16.04 -y
# apt-get install ceph-common -y
# reboot</pre>

You should now be able to connect from an admin workstation and get the nodes in the kubernetes cluster from kubectl (if you do not already have kubectl installed on your admin workstation, see [here][6] for instructions). 

**Note:** if you expand the CSE cluster at any point (add nodes), you will need to repeat this series of commands on each new node in order for it to be able to mount rbd volumes from the ceph cluster.<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png" alt="" class="wp-image-879" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> <figure class="wp-block-image"><img loading="lazy" decoding="async" width="352" height="102" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-20.png" alt="" class="wp-image-856" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-20.png 352w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-20-300x87.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-20-150x43.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-20-250x72.png 250w" sizes="(max-width: 352px) 100vw, 352px" /></figure> 

You should also be able to verify that the core kubernetes services are running in your cluster:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="170" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png" alt="" class="wp-image-879" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn.png 170w, https://kiwicloud.ninja/wp-content/uploads/2019/01/admin-ws-btn-150x28.png 150w" sizes="(max-width: 170px) 100vw, 170px" /> </figure> <figure class="wp-block-image"><img loading="lazy" decoding="async" width="668" height="246" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-21.png" alt="" class="wp-image-857" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-21.png 668w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-21-300x110.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-21-150x55.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-21-250x92.png 250w" sizes="(max-width: 668px) 100vw, 668px" /></figure> 

The ceph configuration files from the ceph cluster nodes need to be added to all nodes in the kubernetes cluster. Depending on which ssh keys you have configured for access, you may be able to do this directly from the ceph admin node as follows:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/ceph-admin-btn.png" alt="" class="wp-image-880" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">$ sudo scp /etc/ceph/ceph.* root@192.168.207.102:/etc/ceph/
$ sudo scp /etc/ceph/ceph.* root@192.168.207.103:/etc/ceph/
$ sudo scp /etc/ceph/ceph.* root@192.168.207.104:/etc/ceph/
$ sudo scp /etc/ceph/ceph.* root@192.168.207.105:/etc/ceph/</pre>

If not, manually copy the /etc/ceph/ceph.conf and /etc/ceph/ceph.client.admin.keyring files to each of the kubernetes nodes using copy/paste or scp from your admin workstation (copy the files from the ceph admin node to ensure that the rbd\_default\_features line is included).

To confirm everything is configured correctly, we should now be able to create and mount a test rbd volume on any of the kubernetes nodes as we did for the ceph admin node previously:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">root@mstr-x4nb:~# rbd create --size 1G kube/testvol02
root@mstr-x4nb:~# rbd ls kube
root@mstr-x4nb:~# rbd info kube/testvol02
rbd image 'testvol02':
        size 1 GiB in 256 objects
        order 22 (4 MiB objects)
        id: 10f36b8b4567
        block_name_prefix: rbd_data.10f36b8b4567
        format: 2
        features: layering, exclusive-lock
        op_features:
        flags:
        create_timestamp: Sun Jan 27 21:56:59 2019
root@mstr-x4nb:~# rbd map kube/testvol02
/dev/rbd0
root@mstr-x4nb:~# mkdir /testmnt
root@mstr-x4nb:~# mkfs.xfs /dev/rbd0
meta-data=/dev/rbd0              isize=512    agcount=9, agsize=31744 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=0
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=1024   swidth=1024 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=8 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
root@mstr-x4nb:~# mount /dev/rbd0 /testmnt
root@mstr-x4nb:~# df -vh
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           395M  5.7M  389M   2% /run
/dev/sda1       9.6G  4.0G  5.6G  42% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/sda15      105M  3.4M  101M   4% /boot/efi
tmpfs           395M     0  395M   0% /run/user/0
/dev/rbd0      1014M   34M  981M   4% /testmnt
root@mstr-x4nb:~# umount /testmnt
root@mstr-x4nb:~# rbd unmap kube/testvol02
root@mstr-x4nb:~# rmdir /testmnt/
root@mstr-x4nb:~# rbd remove kube/testvol02
Removing image: 100% complete...done.</pre>

Note: If the **rbd map** command hangs you may still be running the stock Linux kernel on the kubernetes nodes &#8211; make sure you have restarted them.

Now we have a functional ceph storage cluster capable of serving block storage devices over the network, and a Kubernetes cluster configured able to mount rbd devices and use these. In the next section we will configure kubernetes and ceph together with the rbd-provisioner container to enable dynamic persistent storage for pods deployed into our infrastructure.

## Putting it all together {.wp-block-heading}

### Kubernetes secrets {.wp-block-heading}

We need to first tell Kubernetes account information to be used to connect to the ceph cluster, to do this we create a &#8216;secret&#8217; for the ceph admin user, and also create a client user to be used by k8s provisioning. Working on the kubernetes master node is easiest for this as it has ceph and kubectl already configured from our previous steps:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># ceph auth get-key client.admin</pre>

This will return a key like &#8216;AQCLY0pcFXBYIxAAhmTCXWwfSIZxJ3WhHnqK/w==&#8217; which is used in the next command (**Note:** the &#8216;=&#8217; sign between **&#8211;from-literal** and **key** is **not** a typo &#8211; it actually needs to be like this).<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># kubectl create secret generic ceph-secret --type="kubernetes.io/rbd" \
--from-literal=key='AQCLY0pcFXBYIxAAhmTCXWwfSIZxJ3WhHnqK/w==' --namespace=kube-system
secret "ceph-secret" created</pre>

We can now create a new ceph user &#8216;kube&#8217; and register the secret from this user in kubernetes as &#8216;ceph-secret-kube&#8217;:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># ceph auth get-or-create client.kube mon 'allow r' osd 'allow rwx pool=kube'
[client.kube]
        key = AQDqZU5c0ahCOBAA7oe+pmoLIXV/8OkX7cNBlw==
# kubectl create secret generic ceph-secret-kube --type="kubernetes.io/rbd" \
--from-literal=key='AQDqZU5c0ahCOBAA7oe+pmoLIXV/8OkX7cNBlw==' --namespace=kube-system
secret "ceph-secret-kube" created</pre>

### rbd-provisioner {.wp-block-heading}

Kubernetes is in the process of moving storage provisioners (such as the rbd one we will be using) out of its main packages and into separate projects and packages. There&#8217;s also an issue that the kubernetes-controller-manager container no longer has access to an &#8216;rbd&#8217; binary in order to be able to connect to a ceph cluster directly. We therefore need to deploy a small &#8216;rbd-provisioner&#8217; to act as the go-between from the kubernetes cluster to the ceph storage cluster. This project is available under [this][7] link and the steps below show how to obtain get a kubernetes pod running the rbd-provisioner service up and running (again working from the k8s cluster &#8216;master&#8217; node):<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># git clone https://github.com/kubernetes-incubator/external-storage
Cloning into 'external-storage'...
remote: Enumerating objects: 2, done.
remote: Counting objects: 100% (2/2), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 63661 (delta 0), reused 1 (delta 0), pack-reused 63659
Receiving objects: 100% (63661/63661), 113.96 MiB | 8.97 MiB/s, done.
Resolving deltas: 100% (29075/29075), done.
Checking connectivity... done.
# cd external-storage/ceph/rbd/deploy
# sed -r -i "s/namespace: [^ ]+/namespace: kube-system/g" ./rbac/clusterrolebinding.yaml ./rbac/rolebinding.yaml
# kubectl -n kube-system apply -f ./rbac
clusterrole.rbac.authorization.k8s.io "rbd-provisioner" created
clusterrolebinding.rbac.authorization.k8s.io "rbd-provisioner" created
deployment.extensions "rbd-provisioner" created
role.rbac.authorization.k8s.io "rbd-provisioner" created
rolebinding.rbac.authorization.k8s.io "rbd-provisioner" created
serviceaccount "rbd-provisioner" created
# cd</pre>

You should now be able to see the &#8216;rbd-provisioner&#8217; container starting and then running in kubernetes:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> <figure class="wp-block-image"><img loading="lazy" decoding="async" width="566" height="259" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-22.png" alt="" class="wp-image-860" srcset="https://kiwicloud.ninja/wp-content/uploads/2019/01/image-22.png 566w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-22-300x137.png 300w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-22-150x69.png 150w, https://kiwicloud.ninja/wp-content/uploads/2019/01/image-22-250x114.png 250w" sizes="(max-width: 566px) 100vw, 566px" /></figure> 

### Testing it out {.wp-block-heading}

Now we can create our kubernetes Storageclass using this storage ready for a pod to make a persistent volume claim (PVC) against. Create the following as a new file (I&#8217;ve named mine &#8216;rbd-storageclass.yaml&#8217;). Change the &#8216;monitors&#8217; line to reflect the IP addresses of the &#8216;mon&#8217; nodes in your ceph cluster (in our case these are on the ceph01, ceph02 and ceph03 nodes on the IP addresses shown in the file).<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="shell" data-enlighter-theme="beyond" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rbd
provisioner: ceph.com/rbd
parameters:
  monitors: 192.168.207.201:6789, 192.168.207.202:6789, 192.168.207.203:6789
  adminId: admin
  adminSecretName: ceph-secret
  adminSecretNamespace: kube-system
  pool: kube
  userId: kube
  userSecretName: ceph-secret-kube
  userSecretNamespace: kube-system
  imageFormat: "2"
  imageFeatures: layering</pre>

You can then add this StorageClass to kubernetes using:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># kubectl create -f ./rbd-storageclass.yaml
storageclass.storage.k8s.io "rbd" created</pre>

Next we can create a test PVC and make sure that storage is created in our ceph cluster and assigned to the pod. Create a new file &#8216;pvc-test.yaml&#8217; as:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="beyond" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: testclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: rbd</pre>

We can now submit the PVC to kubernetes and check it has been successfully created:<figure class="wp-block-image">

<img loading="lazy" decoding="async" width="105" height="32" src="https://kiwicloud.ninja/wp-content/uploads/2019/01/k8s-master-btn.png" alt="" class="wp-image-885" /> </figure> 

<pre class="EnlighterJSRAW" data-enlighter-language="msdos" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># kubectl create -f ./pvc-test.yaml
persistentvolumeclaim "testclaim" created
# kubectl get pvc testclaim
NAME        STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
testclaim   Bound     pvc-1e9bdbfd-22a8-11e9-ba77-005056340036   1Gi        RWO            rbd            21s
# kubectl describe pvc testclaim
Name:          testclaim
Namespace:     default
StorageClass:  rbd
Status:        Bound
Volume:        pvc-1e9bdbfd-22a8-11e9-ba77-005056340036
Labels:        &lt;none>
Annotations:   pv.kubernetes.io/bind-completed=yes
               pv.kubernetes.io/bound-by-controller=yes
               volume.beta.kubernetes.io/storage-provisioner=ceph.com/rbd
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
Events:
  Type    Reason                 Age   From                                                                               Message
  ----    ------                 ----  ----                                                                               -------
  Normal  ExternalProvisioning   3m    persistentvolume-controller                                                        waiting for a volume to be created, either by external provisioner "ceph.com/rbd" or manually created by system administrator
  Normal  Provisioning           3m    ceph.com/rbd_rbd-provisioner-bc956f5b4-g6rc2_1f37a6c3-22a6-11e9-aa61-7620ed8d4293  External provisioner is provisioning volume for claim "default/testclaim"
  Normal  ProvisioningSucceeded  3m    ceph.com/rbd_rbd-provisioner-bc956f5b4-g6rc2_1f37a6c3-22a6-11e9-aa61-7620ed8d4293  Successfully provisioned volume pvc-1e9bdbfd-22a8-11e9-ba77-005056340036
# rbd list kube
kubernetes-dynamic-pvc-25e94cb6-22a8-11e9-aa61-7620ed8d4293
# rbd info kube/kubernetes-dynamic-pvc-25e94cb6-22a8-11e9-aa61-7620ed8d4293
rbd image 'kubernetes-dynamic-pvc-25e94cb6-22a8-11e9-aa61-7620ed8d4293':
        size 1 GiB in 256 objects
        order 22 (4 MiB objects)
        id: 11616b8b4567
        block_name_prefix: rbd_data.11616b8b4567
        format: 2
        features: layering
        op_features:
        flags:
        create_timestamp: Mon Jan 28 02:55:19 2019</pre>

As we can see, our test claim has successfully requested and bound a persistent storage volume from the ceph cluster.

## References {.wp-block-heading}

<table class="wp-block-table">
  <tr>
    <td>
      Ceph
    </td>
    
    <td>
      <a href="https://ceph.com/" class="broken_link">https://ceph.com/</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Docker
    </td>
    
    <td>
      <a href="https://www.docker.com/">https://www.docker.com/</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Kubernetes
    </td>
    
    <td>
      <a href="https://kubernetes.io/">https://kubernetes.io/</a>
    </td>
  </tr>
  
  <tr>
    <td>
      VMware Container Service Extension
    </td>
    
    <td>
      <a href="https://vmware.github.io/container-service-extension/">https://vmware.github.io/container-service-extension/</a> 
    </td>
  </tr>
  
  <tr>
    <td>
      VMware vCloud Director for Service Providers
    </td>
    
    <td>
      <a href="https://docs.vmware.com/en/vCloud-Director/index.html">https://docs.vmware.com/en/vCloud-Director/index.html ﻿</a>
    </td>
  </tr>
</table>

Wow, this post ended up way longer than I was anticipating when I started writing it. Hopefully there&#8217;s something useful for you in amongst all of that.

I&#8217;d like to thank members of the vExpert community for their encouragement and advice in getting this post written up and as always, if you have any feedback please leave a comment.

Time-permitting, there will be a followup to this post which details how to deploy containers to this platform using the persistent storage made available, both directly in Kubernetes and using Helm charts. I&#8217;d also like to cover some of the more advanced issues using persistent storage in containers raises &#8211; in particular backup/recovery and replication/high availability of data stored in this manner.

Jon

 [1]: https://kiwicloud.ninja/wp-content/uploads/2019/01/image.png
 [2]: https://kiwicloud.ninja/wp-content/uploads/2019/01/image-1.png
 [3]: https://kiwicloud.ninja/wp-content/uploads/2019/01/image-2.png
 [4]: http://152.67.105.113/2018/02/using-vmware-container-service-extension-cse/
 [5]: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
 [6]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
 [7]: https://github.com/kubernetes-incubator/external-storage/tree/master/ceph/rbd/deploy