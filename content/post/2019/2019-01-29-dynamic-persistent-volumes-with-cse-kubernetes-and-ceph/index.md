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
## Introduction

Application containerization with <a rel="noopener" href="https://www.docker.com/" target="_blank">Docker</a> is fast becoming the default deployment pattern for many business applications and <a rel="noopener" href="https://kubernetes.io/" target="_blank">Kubernetes (k8s)</a> the method of managing these workloads. While containers generally should be stateless and ephemeral (able to be deployed, scaled and deleted at will) almost all business applications require data persistence of some form. In some cases it is appropriate to offload this to an external system (a database, file store or object store in public cloud environments are common for example).

This doesn’t cover all storage requirements though, and if you are running k8s in your own environment or in a hosted service provider environment you may not have access to compatible or appropriate storage. One solution for this is to build a storage platform alongside a Kubernetes cluster which can provide storage persistence while operating in a similar deployment pattern to the k8s cluster itself (scalable, clustered, highly available and no single points of failure).

VMware <a href="https://vmware.github.io/container-service-extension/" target="_blank" rel="noopener">Container Service Extension (CSE)</a> for <a href="https://docs.vmware.com/en/vCloud-Director/index.html" target="_blank" rel="noopener">vCloud Director (vCD)</a> is an automated way for customers of vCloud powered service providers to easily deploy, scale and manage k8s clusters, however CSE currently only provides a limited storage option (an NFS storage server added to the cluster) and k8s persistent volumes (PVs) have to be pre-provisioned in NFS and assigned to containers/pods rather than being generated on-demand. This can also cause availability, scale and performance issues caused by the pod storage being located on a single server VM.

There is certainly no ‘right’ answer to the question of persistent storage for k8s clusters – often the choice will be driven by what is available in the platform you are deploying to and the security, availability and performance requirements for this storage.

In this post I will detail a deployment using a <a rel="noopener" href="https://ceph.com/" target="_blank" class="broken_link">ceph</a> storage cluster to provide a highly available and scalable storage platform and the configuration required to enable a CSE deployed k8s cluster to use dynamic persistent volumes (DPVs) in this environment.

Due to the large number of servers/VMs involved, and the possibility of confusion / working on the wrong server console - I've added buttons like this <img style='vertical-align: middle;' src="./ceph-both.png" /> prior to each section to show which system(s) the commands should be used on.


{{% notice note Disclaimer %}}
I am not an expert in Kubernetes or ceph and have figured out most of the contents in this post from documentation, forums, google and (sometimes) trial and error. Refer to the documentation and support resources at the links at the end of this post if you need the ‘proper’ documentation on these components. Please do not use anything shown in this post in a production environment without appropriate due diligence and making sure you understand what you are doing (and why!).
{{% /notice %}}

With that out of the way - on with the configuration...

## Solution Overview

Our solution is going to be based on a minimal viable installation of ceph with a CSE cluster consisting of 4 ceph nodes (1 admin and 3 combined OSD/mon/mgr nodes) and a 4 node Kubernetes cluster (1 master and 3 worker nodes). There is no requirement for the OS in the ceph cluster and the kubernetes cluster to be the same, however it does make it easier if the packages used for ceph are at the same version which is easier to achieve using the same base OS for both clusters. Since CSE currently only has templates for Ubuntu 16.04 and Photon OS, and due to the lack of packages for the ‘mimic’ release of ceph on Photon OS, this example will use Ubuntu 16.04 LTS as the base OS for all servers.

The diagram below shows the components required to be deployed – in the lab environment I’m using the DNS and NTP servers already exist:
![](solution-overview-network2.png)

{{% notice note Note %}}
In production ceph clusters, the monitor (mon) service should run on separate machines from the nodes providing storage (OSD nodes), but for a test/dev environment there is no issue running both services on the same nodes.
{{% /notice %}}

### Pre-requisites

You should ensure that you have the following enabled and configured in your environment before proceeding:

|Configuration Item|Requirement|
|---|---|
|DNS|Have a DNS server available and add host (‘A’) records for each of the ceph servers. Alternatively it should be possible to add /etc/hosts records on each node to avoid the need to configure DNS. Note that this is only required for the ceph nodes to talk to each other, the kubernetes cluster uses direct IP addresses to contact the ceph cluster.|
|NTP|Have an available NTP time source on your network, or access to external ntp servers|
|Static IP Pool|Container Service Extension (CSE) requires a vCloud OrgVDC network with sufficient addresses available from a static IP pool for the number of kubernetes nodes being deployed|
|SSH Key Pair|Generated SSH key pair to be used to administer the deployed CSE servers. This could (optionally) also be used to administer the ceph servers|
|VDC Capacity|Ensure you have sufficient resources (Memory, CPU, Storage and number of VMs) in your vCD VDC to support the desired cluster sizes|

## Ceph Storage Cluster

The process below describes installing and configuring a ceph cluster on virtualised hardware. If you have an existing ceph cluster available or are building on physical hardware it's best to follow the ceph official documentation at [this link](https://docs.ceph.com/en/reef/) for your circumstances.

### Ceph Server Builds

The 4 ceph servers can be built using any available hardware or virtualisation platform, in this exercise I’ve built them from an Ubuntu 16.04 LTS server template with 2 vCPUs and 4GB RAM for each in the same vCloud Director environment which will be used for deployment of the CSE kubernetes cluster. There are no special requirements for installing/configuring the base Operating System for the ceph cluster. If you are using a different Linux distribution then check the ceph documentation for the appropriate steps for your distribution.

On the 3 storage nodes (ceph01, ceph02 and ceph03) add a hard disk to the server which will act as the storage for the ceph Object Storage Daemon (OSD) – the storage pool which will eventually be useable in Kubernetes. In this example I’ve added a 50GB disk to each of these VMs.

Once the servers are deployed the following are performed on each server to update their repositories and upgrade any modules to current security levels. We will also upgrade the Linux kernel to a more up-to-date version by enabling the Ubuntu Hardware Extension (HWE) kernel which resolves some compatibility issues between ceph and older Linux kernel versions.

<img src="./ceph-both.png" />

```bash
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install --install-recommends linux-generic-hwe-16.04 -y
```

Each server should now be restarted to ensure the new Linux kernel is loaded and any added storage disks are recognised.

### Ceph Admin Account

We need a user account configured on each of the ceph servers to allow ceph-deploy to work and to co-ordinate access, this account must NOT be named 'ceph' due to potential conflicts in the ceph-deploy scripts, but can be called just about anything else. In this lab environment I've used 'cephadmin'. First we create the account on each server and set the password, the 3rd line permits the cephadmin user to use 'sudo' without a password which is required for the ceph-deploy script:

<img src="./ceph-both.png" />

```bash
$ sudo useradd -d /home/cephadmin -m cephadmin -s /bin/bash
$ sudo passwd cephadmin
$ echo "cephadmin ALL = (root) NOPASSWD:ALL" > /etc/sudoers.d/cephadmin
```

From now on, (unless specified) use the new cephadmin login to perform each step. Next we need to generate an SSH key pair for the ceph admin user and copy this to the authorized-keys file on each of the ceph nodes.

Execute the following on the ceph admin node (as cephadmin):

<img src="./ceph-admin-btn.png" />

```bash
$ ssh-keygen -t rsa
```

Accept the default path (`/home/cephadmin/.ssh/id_rsa`) and don't set a key passphrase. You should copy the generated `.ssh/id_rsa` (private key) file to your admin workstation so you can use it to authenticate to the ceph servers.

Next, enable password logins (temporarily) on the storage nodes (ceph01,2 & 3) by running the following on each node:
<img src="./ceph-nodes-btn.png" />

```bash
$ sudo sed -i "s/.*PasswordAuthentication.*/PasswordAuthentication yes/g" /etc/ssh/sshd_config
$ sudo systemctl restart sshd
```

Now copy the cephadmin public key to each of the other ceph nodes by running the following (again only on the admin node):

<img src="./ceph-admin-btn.png" />

```bash
$ ssh-keyscan -t rsa ceph01 >> ~/.ssh/known_hosts
$ ssh-keyscan -t rsa ceph02 >> ~/.ssh/known_hosts
$ ssh-keyscan -t rsa ceph03 >> ~/.ssh/known_hosts
$ ssh-copy-id cephadmin@ceph01
$ ssh-copy-id cephadmin@ceph02
$ ssh-copy-id cephadmin@ceph03
```

You should now confirm you can ssh to each storage node as the cephadmin user from the admin node without being prompted for a password:

<img src="./ceph-admin-btn.png" />

```bash
$ ssh cephadmin@ceph01 sudo hostname
ceph01
$ ssh cephadmin@ceph02 sudo hostname
ceph02
$ ssh cephadmin@ceph03 sudo hostname
ceph03
```

If everything is working correctly then each command will return the appropriate hostname for each storage node without any password prompts.

Optional: It is now safe to re-disable password authentication on the ceph servers if required (since public key authentication will be used from now on) by:

<img src="./ceph-both.png" />

```bash
$ sudo sed -i "s/.*PasswordAuthentication.*/PasswordAuthentication no/g" /etc/ssh/sshd_config
$ sudo systemctl restart sshd
```

You'll need to resolve any authentication issues before proceeding as the ceph-deploy script relies on being able to obtain sudo-level remote access to all of the storage nodes to install ceph successfully.

You should also at this stage confirm that you have time synchronised to an external source on each ceph node so that the server clocks agree, by default on Ubuntu 16.04 timesyncd is configured automatically so nothing needs to be done here in our case. You can check this on Ubuntu 16.04 by running timedatectl:

<img src="./ceph-both.png" />

![Checking time synchronization using timedatectl](image_thumb.png)

For some Linux distributions you may need to create firewall rules at this stage for ceph to function, generally port 6789/tcp (for mon) and the range 6800 to 7300 tcp (for OSD communication) need to be open between the cluster nodes. The default firewall settings in Ubuntu 16.04 allow all network traffic so this is not required (however, do not use this in a production environment without configuring appropriate firewalling).

### Ceph Installation

On all nodes and signed-in as the cephadmin user (**important!**)  
Add the release key: 

<img src="./ceph-both.png" />

```bash
$ wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
```

Add ceph packages to your repository:<figure class="wp-block-image">

<img src="./ceph-both.png" />

```bash
$ echo deb https://download.ceph.com/debian-mimic/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```

On the admin node only, update and install ceph-deploy:

<img src="./ceph-admin-btn.png" />


```bash
$ sudo apt update; sudo apt install ceph-deploy -y
```

On all nodes, update and install ceph-common:

<img src="./ceph-both.png" />
```bash
$ sudo apt update; sudo apt install ceph-common -y
```

{{% notice info Note %}}
Installing ceph-common on the storage nodes isn't strictly required as the ceph-deploy script can do this during cluster initiation, but pre-installing it in this way pulls in several dependencies (e.g. python v2 and associated modules) which can prevent ceph-deploy from running if not present so it is easier to do this way.
{{% /notice %}}

Next again working on the admin node logged in as cephadmin, make a directory to store the ceph cluster configuration files and change to that directory. Note that ceph-deploy will use and write files to the current directory so make sure you are in this folder whenever making changes to the ceph configuration.

<img src="./ceph-admin-btn.png" />

```bash
$ sudo apt install ceph-deploy -y
```

Now we can create the initial ceph cluster from the admin node, use ceph-deploy with the 'new' switch and supply the monitor nodes (in our case all 3 nodes will be both monitors and OSD nodes). Make sure you do NOT use sudo for this command and only run on the admin node:

<img src="./ceph-admin-btn.png" />

```bash
$ ceph-deploy new ceph01 ceph02 ceph03
```

If everything has run correctly you'll see output similar to the following:
![Ceph deployment output](image_thumb-1.png)

Checking the contents of the ~/mycluster/ folder should show the cluster configuration files have been added:

<img src="./ceph-admin-btn.png" />

```bash
$ ls -al ~/mycluster
total 24
drwxrwxr-x 2 cephadmin cephadmin 4096 Jan 25 01:03 .
drwxr-xr-x 5 cephadmin cephadmin 4096 Jan 25 00:57 ..
-rw-rw-r-- 1 cephadmin cephadmin  247 Jan 25 01:03 ceph.conf
-rw-rw-r-- 1 cephadmin cephadmin 7468 Jan 25 01:03 ceph-deploy-ceph.log
-rw------- 1 cephadmin cephadmin   73 Jan 25 01:03 ceph.mon.keyring
```

The ceph.conf file will look something like this:

<img src="./ceph-admin-btn.png" />

```bash
$ cat ~/mycluster/ceph.conf
fsid = 98ca274e-f79b-4092-898a-c12f4ed04544
mon_initial_members = ceph01, ceph02, ceph03
mon_host = 192.168.207.201,192.168.207.202,192.168.207.203
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
```

Run the ceph installation for the nodes (again from the admin node only):

<img src="./ceph-admin-btn.png" />

```bash
$ ceph-deploy install ceph01 ceph02 ceph03
```

This will run through the installation of ceph and pre-requisite packages on each node, you can check the ceph-deploy-ceph.log file after deployment for any issues or errors.

### Ceph Configuration

Once you've successfully installed ceph on each node, use the following (again from only the admin node) to deploy the initial ceph monitor services:

<img src="./ceph-admin-btn.png" />

```bash
$ ceph-deploy mon create-initial
```

If all goes well you'll get some messages at the completion of this process showing the keyring files being stored in your 'mycluster' folder, you can check these exist:

<img src="./ceph-admin-btn.png" />

```bash
$ ls -al ~/mycluster
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
-rw------- 1 cephadmin cephadmin     73 Jan 25 01:03 ceph.mon.keyring
```

To avoid having to specify the monitor node address and ceph.client.admin.keyring path in every command, we can now deploy these to each node so they are available automatically. Again working from the 'mycluster' folder on the admin node:

<img src="./ceph-admin-btn.png" />

```bash
$ ceph-deploy admin cephadmin ceph01 ceph02 ceph03
```

This should give the following:
![Ceph deployment output](image_thumb-2.png)

Next we need to deploy the manager ('mgr') service to the OSD nodes, again working from the 'mycluster' folder on the admin node:

```bash
$ ceph-deploy mgr create ceph01 ceph02 ceph03
```

At this stage we can check that all of the mon and mgr services are started and ok by running (on the admin node):

<img src="./ceph-admin-btn.png" />

```bash
$ sudo ceph -s
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
    pgs:
```

As you can see, the manager ('mgr') service is installed on all 3 nodes but only active on the first and in standby mode on the other 2 - this is normal and correct. The monitor ('mon') service is running on all of the storage nodes.

Next we can configure the disks attached to our storage nodes for use by ceph. Ensure that you know and use the correct identifier for your disk devices (in this case, we are using the 2nd SCSI disk attached to the storage node VMs which is at /dev/sdb so that's what we'll use in the commands below). As before, run the following only on the admin node:

<img src="./ceph-admin-btn.png" />

```bash
$ ceph-deploy osd create --data /dev/sdb ceph01
$ ceph-deploy osd create --data /dev/sdb ceph02
$ ceph-deploy osd create --data /dev/sdb ceph03
```

For each command the last line of the logs shown when run should be similar to 'Host ceph01 is now ready for osd use.'

We can now check the overall cluster health with:

<img src="./ceph-admin-btn.png" />

```bash
$ ssh ceph01 sudo ceph health
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
    pgs:
```

As you can see, the 3 x 50GB disks have now been added and the total (150 GiB) capacity is available under the data: section.

Now we need to create a ceph storage pool ready for Kubernetes to consume from - the default name of this pool is 'rbd' (if not specified), but it is strongly recommended to name it differently from the default when using for k8s so I've created a storage pool called 'kube' in this example (again running from the mycluster folder on the admin node):

<img src="./ceph-admin-btn.png" />

```bash
$ sudo ceph osd pool create kube 30 30
pool 'kube' created
```

The two '30's are important - you should review the ceph documentation <a href="http://docs.ceph.com/docs/mimic/rados/configuration/pool-pg-config-ref" target="_blank" rel="noopener" class="broken_link">here</a> for Pool, PG and CRUSH configuration to establish values for PG and PGP appropriate to your environment.

We now associated this pool with the rbd (RADOS block device) application so it is available to be used as a RADOS block device:

<img src="./ceph-admin-btn.png" />

```bash
$ sudo ceph osd pool application enable kube rbd
enabled application 'rbd' on pool 'kube'
```

### Testing Ceph Storage

The easiest way to test our ceph cluster is working correctly and can provide storage is to attempt creating and using a new RADOS Block Device (rbd) volume from our admin node.

Before this will work we need to tune the rbd features map by editing ceph.conf on our client to disable rbd features that aren't available in our Linux kernel (on admin/client node):

<img src="./ceph-admin-btn.png" />

```bash
$ echo "rbd_default_features = 7" | sudo tee -a /etc/ceph/ceph.conf
rbd_default_features = 7
```

Now we can test creating a volume:

<img src="./ceph-admin-btn.png" />

```bash
$ sudo rbd create --size 1G kube/testvol01
```

Confirm that the volume exists:<figure class="wp-block-image">

<img src="./ceph-admin-btn.png" />

```bash
$ sudo rbd ls kube
testvol01
```

Get information on our volume:

<img src="./ceph-admin-btn.png" />

```bash
$ sudo rbd info kube/testvol01
rbd image 'testvol01':
        size 1 GiB in 256 objects
        order 22 (4 MiB objects)
        id: 10e96b8b4567
        block_name_prefix: rbd_data.10e96b8b4567
        format: 2
        features: layering, exclusive-lock
        op_features:
        flags:
        create_timestamp: Sun Jan 27 08:50:45 2019
```

Map the volume to our admin host (which creates the block device /dev/rbd0):

<img src="./ceph-admin-btn.png" />

```bash
$ sudo rbd map kube/testvol01
/dev/rbd0
```

Now we can create a temporary mount folder, make a filesystem on our volume and mount it to our temporary mount:

<img src="./ceph-admin-btn.png" />

```bash
$ sudo mkdir /testmnt
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
/dev/rbd0      1014M   34M  981M   4% /testmnt
```

We can see our volume has been mounted successfully and can now be used as any other disk.

To tidy up and remove our test volume:

<img src="./ceph-admin-btn.png" />

```bash
$ sudo umount /dev/rbd0
$ sudo rbd unmap kube/testvol01
$ sudo rbd remove kube/testvol01
Removing image: 100% complete...done.
$ sudo rmdir /testmnt
```

## Kubernetes CSE Cluster

Using VMware Container Service Extension (CSE) makes it easy to deploy and configure a base Kubernetes cluster into our vCloud Director platform. I previously wrote a post [here][4] with a step-by-step guide to using CSE.

First we need an ssh key pair to provide to the CSE nodes as they are deployed to allow us to access them. You could re-use the cephadmin key-pair created in the previous section, or generate a new set. As I'm using Windows as my client OS I used the **puttygen** utility included in the [PuTTY][5] package to generate a new keypair and save them to a .ssh directory in my home folder.

{{% notice warning Important %}}
Check your public key file in a text editor prior to deploying the cluster, if it looks like this:
![Public key as generated by PuTTYGen (incorrect)](image-12.png)
You will need to change it to be all on one line starting 'ssh-rsa' and with none of the extra text as follows:
![Public key fixed](image-11.png)
If you do not make this change this you won't be able to authenticate to your cluster nodes once deployed.
{{% /notice %}}

Next we login to vCD using the vcd-cli (see my post linked above if you need to install/configure vcd-cli and the CSE extension):

<img src="./admin-ws-btn.png" />

![Logging in to vcd-cli](image-4.png)

Now we can see what virtual Datacenters (VDCs) are available to us:

<img src="./admin-ws-btn.png" />

![Showing available VDCs](image-5.png)

If we had multiple VDCs available, we need to select which one is 'in_use' (active) for deployment of our cluster using `vcd vdc use "<VDC Name>"`. In this case we only have a single VDC and it's already active/in use.

We can get the information of our VDC which will help us fill out the required properties when creating our k8s cluster:

<img src="./admin-ws-btn.png" />

![VDC Properties returned by vdc info](image-6.png)

We will be using the 'Tyrell Servers A03' network (where our ceph cluster exists) and the 'A03 VSAN Performance' storage profile for our cluster.

To get the options available when creating a cluster we can see the cluster creation help:

<img src="./admin-ws-btn.png" />

![CSE cluster create options](image-7.png)

Now we can go ahead and create out Kubernetes cluster with CSE:

<img src="./admin-ws-btn.png" />

![CSE cluster create](image-17.png)

Looking in vCloud Director we can see the new vApp and VMs deployed:
![CSE Cluster viewed from VCD](image-18.png)

We obtain the kubectl config of our cluster and store this for later use (make the .kube folder first if it doesn't already exist):

<img src="./admin-ws-btn.png" />

```cmd
C:\Users\admin>vcd cse cluster config k8sceph > .kube\config
```

And get the details of our k8s nodes from vcd-cli:

<img src="./admin-ws-btn.png" />

![CSE cluster node details](image-19.png)

Next we need to update and install the ceph client on each cluster node - run the following on each node (including the master). To do this we can connect via ssh as root using the key pair we specified when creating the cluster.

<img src="./k8s-both-btn.png" />

```bash
# wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
OK
# echo deb https://download.ceph.com/debian-mimic/ $(lsb_release -sc) main > /etc/apt/sources.list.d/ceph.list
# apt-get update
# apt-get install --install-recommends linux-generic-hwe-16.04 -y
# apt-get install ceph-common -y
# reboot
```

You should now be able to connect from an admin workstation and get the nodes in the kubernetes cluster from kubectl (if you do not already have kubectl installed on your admin workstation, see [here][6] for instructions).

{{% notice note "Expanding CSE Cluster" %}}
If you expand the CSE cluster at any point (add nodes), you will need to repeat this series of commands on each new node in order for it to be able to mount rbd volumes from the ceph cluster.
{{% /notice %}}

<img src="./admin-ws-btn.png" />

![Showing cluster nodes from kubectl](image-20.png)

You should also be able to verify that the core kubernetes services are running in your cluster:

<img src="./admin-ws-btn.png" />

![Showing cluster pods from kubectl](image-21.png)

The ceph configuration files from the ceph cluster nodes need to be added to all nodes in the kubernetes cluster. Depending on which ssh keys you have configured for access, you may be able to do this directly from the ceph admin node as follows:

<img src="./ceph-admin-btn.png" />

```bash
$ sudo scp /etc/ceph/ceph.* root@192.168.207.102:/etc/ceph/
$ sudo scp /etc/ceph/ceph.* root@192.168.207.103:/etc/ceph/
$ sudo scp /etc/ceph/ceph.* root@192.168.207.104:/etc/ceph/
$ sudo scp /etc/ceph/ceph.* root@192.168.207.105:/etc/ceph/
```

If not, manually copy the `/etc/ceph/ceph.conf` and `/etc/ceph/ceph.client.admin.keyring` files to each of the kubernetes nodes using copy/paste or scp from your admin workstation (copy the files from the ceph admin node to ensure that the `rbd_default_features` line is included).

To confirm everything is configured correctly, we should now be able to create and mount a test rbd volume on any of the kubernetes nodes as we did for the ceph admin node previously:

<img src="./k8s-master-btn.png" />


```bash
root@mstr-x4nb:~# rbd create --size 1G kube/testvol02
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
Removing image: 100% complete...done.
```

{{% notice note "RBD Hanging" %}}
If the **rbd map** command hangs you may still be running the stock Linux kernel on the kubernetes nodes - make sure you have restarted them.
{{% /notice %}}

Now we have a functional ceph storage cluster capable of serving block storage devices over the network, and a Kubernetes cluster configured able to mount rbd devices and use these. In the next section we will configure kubernetes and ceph together with the rbd-provisioner container to enable dynamic persistent storage for pods deployed into our infrastructure.

## Putting it all together

### Kubernetes secrets

We need to first tell Kubernetes account information to be used to connect to the ceph cluster, to do this we create a 'secret' for the ceph admin user, and also create a client user to be used by k8s provisioning. Working on the kubernetes master node is easiest for this as it has ceph and kubectl already configured from our previous steps:

<img src="./k8s-master-btn.png" />


```bash
# ceph auth get-key client.admin
```

This will return a key like `AQCLY0pcFXBYIxAAhmTCXWwfSIZxJ3WhHnqK/w==` which is used in the next command

{{% notice warning Note %}}
The '=' sign between **-from-literal** and **key** in the following command is **not** a typo - it actually needs to be like this.
{{% /notice %}}

<img src="./k8s-master-btn.png" />

```bash
# kubectl create secret generic ceph-secret --type="kubernetes.io/rbd" \
--from-literal=key='AQCLY0pcFXBYIxAAhmTCXWwfSIZxJ3WhHnqK/w==' --namespace=kube-system
secret "ceph-secret" created
```

We can now create a new ceph user 'kube' and register the secret from this user in kubernetes as 'ceph-secret-kube':

<img src="./k8s-master-btn.png" />


```bash
# ceph auth get-or-create client.kube mon 'allow r' osd 'allow rwx pool=kube'
[client.kube]
        key = AQDqZU5c0ahCOBAA7oe+pmoLIXV/8OkX7cNBlw==
# kubectl create secret generic ceph-secret-kube --type="kubernetes.io/rbd" \
--from-literal=key='AQDqZU5c0ahCOBAA7oe+pmoLIXV/8OkX7cNBlw==' --namespace=kube-system
secret "ceph-secret-kube" created
```

### rbd-provisioner

Kubernetes is in the process of moving storage provisioners (such as the rbd one we will be using) out of its main packages and into separate projects and packages. There's also an issue that the kubernetes-controller-manager container no longer has access to an 'rbd' binary in order to be able to connect to a ceph cluster directly. We therefore need to deploy a small 'rbd-provisioner' to act as the go-between from the kubernetes cluster to the ceph storage cluster. This project is available under [this][7] link and the steps below show how to obtain get a kubernetes pod running the rbd-provisioner service up and running (again working from the k8s cluster 'master' node):

<img src="./k8s-master-btn.png" />

```bash
# git clone https://github.com/kubernetes-incubator/external-storage
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
# cd
```

You should now be able to see the 'rbd-provisioner' container starting and then running in kubernetes:

<img src="./k8s-master-btn.png" />

![rbd-provisioner container](image-22.png)

### Testing it out

Now we can create our kubernetes Storageclass using this storage ready for a pod to make a persistent volume claim (PVC) against. Create the following as a new file (I've named mine 'rbd-storageclass.yaml'). Change the 'monitors' line to reflect the IP addresses of the 'mon' nodes in your ceph cluster (in our case these are on the ceph01, ceph02 and ceph03 nodes on the IP addresses shown in the file).

<img src="./k8s-master-btn.png" />


```yaml
apiVersion: storage.k8s.io/v1
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
  imageFeatures: layering
```

You can then add this StorageClass to kubernetes using:

<img src="./k8s-master-btn.png" />


```bash
# kubectl create -f ./rbd-storageclass.yaml
storageclass.storage.k8s.io "rbd" created
```

Next we can create a test PVC and make sure that storage is created in our ceph cluster and assigned to the pod. Create a new file `pvc-test.yaml` as:

<img src="./k8s-master-btn.png" />

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: testclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: rbd
```
We can now submit the PVC to kubernetes and check it has been successfully created:

<img src="./k8s-master-btn.png" />

```bash
# kubectl create -f ./pvc-test.yaml
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
Labels:        <none>
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
        create_timestamp: Mon Jan 28 02:55:19 2019
```

As we can see, our test claim has successfully requested and bound a persistent storage volume from the ceph cluster.

## References

|Item|Link|
|---|---|
|Ceph| [https://ceph.io/](https://ceph.io/) |
|Docker|[https://www.docker.com/](https://www.docker.com/)|
|Kubernetes|[https://kubernetes.io/](https://kubernetes.io/)|
|VMware Container Service Extension|[https://vmware.github.io/container-service-extension/](https://vmware.github.io/container-service-extension/)|
|VMware vCloud Director for Service Providers|[https://docs.vmware.com/en/vCloud-Director/index.html](https://docs.vmware.com/en/vCloud-Director/index.html)|

Wow, this post ended up way longer than I was anticipating when I started writing it. Hopefully there's something useful for you in amongst all of that.

I'd like to thank members of the vExpert community for their encouragement and advice in getting this post written up and as always, if you have any feedback please leave a comment.

Time-permitting, there will be a followup to this post which details how to deploy containers to this platform using the persistent storage made available, both directly in Kubernetes and using Helm charts. I'd also like to cover some of the more advanced issues using persistent storage in containers raises - in particular backup/recovery and replication/high availability of data stored in this manner.

Jon

 [4]: /2018/02/using-vmware-container-service-extension-cse/
 [5]: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
 [6]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
 [7]: https://github.com/kubernetes-incubator/external-storage/tree/master/ceph/rbd/deploy
