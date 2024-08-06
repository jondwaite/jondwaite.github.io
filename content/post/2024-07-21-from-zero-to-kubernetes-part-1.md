---
title: From zero to Kubernetes on Photon OS 5 – Part 1
author: Jon Waite
type: post
date: -001-11-30T00:00:00+00:00
draft: true
url: /?p=69306
series: from-zero-to-kubernetes
categories:
  - Containers
  - Kubernetes
  - Packer
  - Photon OS
  - PowerShell
tags:
  - Automation
  - Containerisation
  - Kubernetes
  - Packer
  - Photon OS
  - PowerCLI
  - PowerShell

---
Part 1 - Build the VM template (this post)

This post is part of a series to deploying and configuring Kubernetes using the latest component versions on VMware Photon OS v5, as I write up additional parts they will be linked in the table above.

## Introduction {#introduction.wp-block-heading}

As I use Kubernetes (k8s) more frequently, I needed to learn more about the latest features (more recent than those typically available from most package managers). I also wanted to see how hard it would be to build a minimal OS template image suitable for experimenting with k8s cluster configurations based on VMware Photon OS 5. This post (and the accompanying [github][1] repository) are a snapshot of my progress so far towards this.

My main goals for this project were to:

<ul style="font-size:15px" class="wp-block-list">
  <li class="has-medium-font-size">
    Build on Photon OS 5.0 minimal installation as the base operating system
  </li>
  <li class="has-medium-font-size">
    Use an automated VM template for easy repeatable deployments
  </li>
  <li class="has-medium-font-size">
    Use latest Kubernetes v1.30.2 / Containerd v1.7.19 / runc v1.1.13 components
  </li>
  <li class="has-medium-font-size">
    Use kubeadm to bootstrap the cluster
  </li>
  <li class="has-medium-font-size">
    Use kube-vip for High Availability (HA) of the kubernetes control-plane
  </li>
  <li class="has-medium-font-size">
    Use kube-vip-cloud-controller to provide load balancing services to other applications
  </li>
  <li class="has-medium-font-size">
    Use calico for CNI networking
  </li>
  <li class="has-medium-font-size">
    Support persistent storage for deployed applications (PVC) on NFS storage
  </li>
  <li class="has-medium-font-size">
    Support deployment of helm charts
  </li>
</ul>

<ul style="font-size:15px" class="wp-block-list">
</ul>

## Design {#design.wp-block-heading}

I wanted to keep the deployment size as minimal as possible while still being 'large' enough to be useful, I settled on a sizing of 3 control-plane and 3 worker nodes for kubernetes each with 2 vCPU cores, 8GB RAM and 16 GB local storage each, the number and sizing of nodes can be easily adjusted as required.

In addition I provisioned a NFS share to act as persistent storage. This entire deployment fits comfortably on a small form-factor PC which I've upgraded to 64 GB RAM and 1 TB NVMe drive The PC is running VMware ESXi as the hypervisor host, but this process could be easily followed using any hypervisor. Performance so far seems extremely good even though the PC only has a 4-core i5-6500T CPU.

### Network Design {#network-design.wp-block-heading}

I defined the following networks to be used for the deployment (using [RFC 1918][2] private IP ranges). The ranges were chosen to provide plenty of address space for my use case and to allow 2nd (and subsequent) clusters to live within a reasonably contained address space (This layout gives me the ability to create up to 32 complete clusters in a single /15 network subnet - 172.30.0.0/15 in my case).<figure class="wp-block-table is-style-stripes has-small-font-size">

| Function         | CIDR          | Start of range | End of range  | Addresses | Notes                                                  |
| ---------------- | ------------- | -------------- | ------------- | --------- | ------------------------------------------------------ |
| Kubernetes Nodes | 172.30.0.0/24 | 172.30.0.0     | 172.30.0.255  | 256       | Used for cluster host VMs and `kube-vip` VIP           |
| Load Balancer    | 172.30.1.0/24 | 172.30.1.0     | 172.30.1.255  | 256       | kube-vip IP pool for load-balanced exposed services    |
| reserved         | 172.30.2.0/23 | 172.30.2.0     | 172.30.3.255  | 512       | Available for any future requirements for this cluster |
| Services network | 172.30.4.0/22 | 172.30.4.0     | 172.30.7.255  | 1024      | Internal services network                              |
| Pod network      | 172.30.8.0/21 | 172.30.8.0     | 172.30.15.255 | 2048      | Internal pod (container) addresses                     |</figure> 

### Host Design {#host-design.wp-block-heading}

Three control-plane nodes and three worker nodes will be deployed in the cluster as in the table below, also shown are the 'cluster' IP address for `kube-vip` and the NFS server used to provide persistent storage:<figure class="wp-block-table is-style-stripes has-small-font-size">

| Hostname    | IP Address  | Subnet mask       | Gateway    | Notes                                               |
| ----------- | ----------- | ----------------- | ---------- | --------------------------------------------------- |
| k8s01-nfs01 | 172.30.0.5  | 255.254.0.0 (/15) | 172.30.0.1 | NFS Server for persistent storage                   |
| k8s01       | 172.30.0.10 | 255.254.0.0 (/15) | 172.30.0.1 | 'Floating' HA IP address for `kube-vip` |
| k8s01-c01   | 172.30.0.11 | 255.254.0.0 (/15) | 172.30.0.1 | Control plane node #1                               |
| k8s01-c02   | 172.30.0.12 | 255.254.0.0 (/15) | 172.30.0.1 | Control plane node #2                               |
| k8s01-c03   | 172.30.0.13 | 255.254.0.0 (/15) | 172.30.0.1 | Control plane node #3                               |
| k8s01-w01   | 172.30.0.14 | 255.254.0.0 (/15) | 172.30.0.1 | Worker node #1                                      |
| k8s01-w02   | 172.30.0.15 | 255.254.0.0 (/15) | 172.30.0.1 | Worker node #2                                      |
| k8s01-w03   | 172.30.0.16 | 255.254.0.0 (/15) | 172.30.0.1 | Worker node #3                                      |</figure> 

I have a local DNS environment so just added all of the host details here to my primary DNS domain as 'A' records (including the `k8s01` and `k8s01-nfs01` names). The same effect can probably be achieved by creating `/etc/hosts` entries for each address on each deployed server (ideally in the template VM so these are automatically present).

## Automation {.wp-block-heading}

It quickly became apparent that the best way to obtain reproducible cluster deployments would be to automate the build of a template VM from which cluster VMs could be cloned and deployed. For those wishing to just use the automation scripting that I developed, a link to the github repository is [here][3]

The sections below detail the actions performed by the automation build, together with some of the reasons for choices made in the configurations.

## Template VM Build {#template-vm-build.wp-block-heading}

For those wanting to build or customise the template image themselves, the steps are included below.

### Pre-requisites: {#pre-requisites.wp-block-heading}

Build a new VM to act as the template for deploying the cluster - this VM will not be used directly but will be cloned to each of the require control-plane and worker nodes later. Specs are based on my experience and work fine - you will probably be able to get away with smaller sizing if CPU and/or RAM are limited:

<ul class="wp-block-list">
  <li>
    Install a new VM from photon 5 minimal ISO image (2 vCPU/8 GB vRAM/16 GB Disk using 'Thin' provisioning)
  </li>
  <li>
    'template' name (e.g. k8s-tmpl) for hostname
  </li>
  <li>
    Use DHCP networking (for now) - use a network supporting DHCP clients that can reach the internet
  </li>
  <li>
    Set initial root password during install and complete initial Photon OS install / remove ISO attachment & reboot after install
  </li>
  <li>
    Login as root on console and check the DHCP IP address assigned (<code>ip a</code>)
  </li>
</ul>

### Enable SSH for root {#enable-ssh-for-root.wp-block-heading}

Many of the steps below are easier to copy/paste to a terminal session rather than retyping on the VM console - to allow external SSH session (e.g. from your desktop), the following allows SSH connections to the 'root' user.

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">sed -i 's/PermitRootLogin no/PermitRootLogin yes/' /etc/ssh/sshd_config
systemctl restart sshd</pre>

You may find it easier at this point to ssh to the template VM and do the remainder of the configuration from a SSH client (e.g. puTTY or similar).

### Configure Firewall {#add-firewall-rules.wp-block-heading}

Kubernetes and addon services need to be able to communicate between the deployed control-plane and worker nodes, the following ports are required (except icmp which I just include for convenience/testing connectivity):<figure class="wp-block-table is-style-stripes">

<table>
  <tr>
    <th class="has-text-align-center" data-align="center">
      Port
    </th>
    
    <th class="has-text-align-center" data-align="center">
      Protocol
    </th>
    
    <th class="has-text-align-center" data-align="center">
      Control-Plane<br />Nodes
    </th>
    
    <th class="has-text-align-center" data-align="center">
      Worker<br />Nodes
    </th>
    
    <th>
      Description
    </th>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      179
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td>
      <code>calico</code> BGP networking
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      2379
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      No
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td>
      <code>etcd</code> & kube-apiserver
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      2380
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      No
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td>
      <code>etcd</code> & kube-apiserver
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      4789
    </td>
    
    <td class="has-text-align-center" data-align="center">
      udp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td>
      <code>calico</code> networking with VXLAN enabled
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      5473
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td>
      <code>calico</code> networking with Typha enabled
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      6443
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td class="has-text-align-center" data-align="center">
      No
    </td>
    
    <td>
      Kubernetes API server
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      10250
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td>
      <code>kubelet</code> API server
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      10256
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td class="has-text-align-center" data-align="center">
      No
    </td>
    
    <td>
      Kubernetes Load balancers
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      10257
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      No
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td>
      <code>kube-controller-manager</code>
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      10259
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      No
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td>
      <code>kube-scheduler</code>
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      30000-32767
    </td>
    
    <td class="has-text-align-center" data-align="center">
      tcp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes*
    </td>
    
    <td class="has-text-align-center" data-align="center">
      No*
    </td>
    
    <td>
      NodePort Services
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      all
    </td>
    
    <td class="has-text-align-center" data-align="center">
      ip-in-ip
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td class="has-text-align-center" data-align="center">
      Yes
    </td>
    
    <td>
      <code>calico</code> IP-in-IP
    </td>
  </tr>
  
  <tr>
    <td class="has-text-align-center" data-align="center">
      all
    </td>
    
    <td class="has-text-align-center" data-align="center">
      icmp
    </td>
    
    <td class="has-text-align-center" data-align="center">
      No
    </td>
    
    <td class="has-text-align-center" data-align="center">
      No
    </td>
    
    <td>
      Allows <code>ping</code> between cluster nodes
    </td>
  </tr>
</table></figure> 

<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
  <p>
    <strong>NOTE:</strong> *This is the default port range used by NodePort services (it can be changed) and if the control-plane servers are 'untainted' (to allow them to run workloads) will also be required on control-plane nodes. In our deployment we're not planning on using NodePort services so this could safely be omitted.
  </p>
</blockquote>

The rules cover both control-plane and worker node requirements (both sets are included to simplify template) and to save them to be persistent across host restarts:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">iptables -A INPUT -p tcp --dport 179 -j ACCEPT
iptables -A INPUT -p tcp --dport 2379:2380 -j ACCEPT
iptables -A INPUT -p udp --dport 4789 -j ACCEPT
iptables -A INPUT -p tcp --dport 5473 -j ACCEPT
iptables -A INPUT -p tcp --dport 6443 -j ACCEPT
iptables -A INPUT -p udp --dport 8285 -j ACCEPT
iptables -A INPUT -p udp --dport 8472 -j ACCEPT
iptables -A INPUT -p tcp --dport 10250 -j ACCEPT
iptables -A INPUT -p tcp --dport 10256 -j ACCEPT
iptables -A INPUT -p tcp --dport 10257 -j ACCEPT
iptables -A INPUT -p tcp --dport 10259 -j ACCEPT
iptables -A INPUT -p tcp --dport 30000:32767 -j ACCEPT
iptables -A INPUT -p ip -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables-save > /etc/systemd/scripts/ip4save</pre>

### Configure ssh keys {#configure-ssh-keys.wp-block-heading}

<blockquote class="wp-block-quote is-layout-flow wp-block-quote-is-layout-flow">
  <p>
    <strong>!!WARNING!!</strong> Setting up ssh keys in the template in the way described here will allow all deployed VMs from the template to ssh (to each other) without passwords. Obviously this is a huge security risk and combined with enabling the ssh service for 'root' shouldn't be used anywhere near a public network.
  </p>
</blockquote>

Generate public/private key pair:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">ssh-keygen -t rsa -b 4096 -N '' <<< $'\ny' >/dev/null 2>&1</pre>

Add public key to athorized_keys file (so that each deployed server will trust the others automatically)

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys</pre>

### Configure kernel modules {#configure-kernel-modules.wp-block-heading}

Add the `overlay` and `br_netfilter` kernel modules to be loaded automatically (they will be loaded after the reboot after the next step):

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">echo overlay > /etc/modules-load.d/20-containerd.conf
echo br_netfilter >> /etc/modules-load.d/20-containerd.conf</pre>

### Package removes/adds/updates {#package-removesaddsupdates.wp-block-heading}

The latest version of containerd in Photon OS installed by tdnf updates is from May 2023, remove and replace by current release (at time of writing) version and remove docker and associated files:

Requires internet connectivity for the template VM, reboot to ensure updated kernel & modules loaded as well as aliases for containerd are removed:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">tdnf remove -y docker* containerd* runc
tdnf install -y wget tar jq socat ethtool conntrack git nfs-utils
tdnf update -y
reboot</pre>

Once the VM has restarted, reconnect via ssh as root and continue.

### Configure kernel settings {#configure-kernel-settings.wp-block-heading}

The pre-defined `50-security-hardening.conf` file disables ipv4 forwarding so make sure we are after that in the execution order by setting a higher value:

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">echo net.ipv4.ip_forward=1 > /etc/sysctl.d/60-kubernetes.conf
sysctl --system</pre>

### (Optional) Make a downloads folder and cd into it {#optional-make-a-downloads-folder-and-cd-into-it.wp-block-heading}

Not required, but this way all of the downloads can be easily cleaned up prior to turning the VM into a template to reduce overall storage

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">mkdir ~/k8s-downloads
cd ~/k8s-downloads
</pre>

### Install containerd {#install-containerd.wp-block-heading}

Check [here][4] for latest release and copy link then install:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">wget https://github.com/containerd/containerd/releases/download/v1.7.19/containerd-1.7.19-linux-amd64.tar.gz
tar Czxvf /usr/local containerd-1.7.19-linux-amd64.tar.gz
wget -P /usr/local/lib/systemd/system https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
systemctl daemon-reload
systemctl enable --now containerd
</pre>

### Install updated runc {#install-updated-runc.wp-block-heading}

Not sure if this is necessary - but figured can't hurt to have latest release from [here][5] then install (this is needed if original runc was removed earlier):

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">wget https://github.com/opencontainers/runc/releases/download/v1.1.13/runc.amd64
install runc.amd64 -o root -g root -m 0755 /usr/bin/runc
</pre>

### Install CNI plugins {#install-cni-plugins.wp-block-heading}

Check [here][6] and grab latest CNI plugins package and install:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">wget https://github.com/containernetworking/plugins/releases/download/v1.5.1/cni-plugins-linux-amd64-v1.5.1.tgz
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.1.tgz
</pre>

### Configure containerd {#configure-containerd.wp-block-heading}

Generate default containerd configuration file, then edit to enable systemd Cgroups and use an updated `pause` image to prevent warnings from `kubeadm`:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">/usr/local/bin/containerd config default > /etc/containerd/config.toml
sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/' /etc/containerd/config.toml
sed -i 's/pause\:3.8/pause\:3.9/' /etc/containerd/config.toml
</pre>

### Install nerdctl (Optional) {#install-nerdctl-optional.wp-block-heading}

Check releases [here][7] and grab 'latest' and install:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">wget https://github.com/containerd/nerdctl/releases/download/v1.7.6/nerdctl-1.7.6-linux-amd64.tar.gz
tar Cxzvf /usr/local/bin nerdctl-1.7.6-linux-amd64.tar.gz
</pre>

Provides nerdctl which can be useful to troubleshoot kubernetes containers:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">nerdctl --namespace k8s.io ps -a
</pre>

### Install kubernetes binaries: {#install-kubernetes-binaries.wp-block-heading}

Pre-stage all of the binaries and service definitions in the template so they're ready to go:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group=""># kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# kubectl-convert
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert"
install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert 

# crictl
curl -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.30.1/crictl-v1.30.1-linux-amd64.tar.gz
tar Czxvf /usr/local/bin crictl-v1.30.1-linux-amd64.tar.gz

# kubeadm and kubelet
curl -L --remote-name-all https://dl.k8s.io/release/v1.30.2/bin/linux/amd64/{kubeadm,kubelet}
install -o root -g root -m 0755 kubeadm /usr/local/bin/kubeadm
install -o root -g root -m 0755 kubelet /usr/local/bin/kubelet

# Configure systemd services
wget -P /usr/local/lib/systemd/system https://raw.githubusercontent.com/kubernetes/release/master/cmd/krel/templates/latest/kubelet/kubelet.service
sed -i 's/\/usr\/bin\/kubelet/\/usr\/local\/bin\/kubelet/' /usr/local/lib/systemd/system/kubelet.service
mkdir -p /usr/local/lib/systemd/system/kubelet.service.d
wget -P /usr/local/lib/systemd/system/kubelet.service.d https://raw.githubusercontent.com/kubernetes/release/master/cmd/krel/templates/latest/kubeadm/10-kubeadm.conf
sed -i 's/\/usr\/bin\/kubelet/\/usr\/local\/bin\/kubelet/' /usr/local/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload
systemctl enable --now kubelet
kubeadm config images pull
</pre>

### Setup bash aliases & completion {#setup-bash-aliases--completion.wp-block-heading}

This setups up bash completion for `kubectl` and also allows `k` to be used (instead of `kubectl`) as an alias:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">cat <<EOF >> $HOME/.profile
if [ -n "$BASH_VERSION" ]; then
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi
EOF
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
source $HOME/.bashrc
</pre>

### Tidy up and prepare template for cloning {#tidy-up-and-prepare-template-for-cloning.wp-block-heading}

We can now remove all of the downloaded files and do the following to ensure that the ssh keys and machine-id are recreated when the template is cloned, we then shutdown the template VM:

<pre class="EnlighterJSRAW" data-enlighter-language="bash" data-enlighter-theme="atomic" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">cd ~
rm -rf ~/k8s-downloads
rm -rf /etc/ssh/*_key
rm -rf /etc/ssh/*pub
echo "" > /etc/machine-id
shutdown -h now
</pre>

Now convert the template VM to a template (or upload to a content library) ready to be cloned/deployed to build our actual cluster VMs.

 [1]: https://github.com/jondwaite/k8s-photon
 [2]: https://datatracker.ietf.org/doc/html/rfc1918
 [3]: https://github.com/jondwaite/k8s-photon/
 [4]: https://github.com/containerd/containerd/releases
 [5]: https://github.com/opencontainers/runc/releases
 [6]: https://github.com/containernetworking/plugins/releases
 [7]: https://github.com/containerd/nerdctl/releases/