---
title: From zero to Kubernetes on Photon OS 5 – Part 1
author: Jon Waite
type: post
date: 2024-07-21T09:20:27+00:00
draft: true
url: /2024/07/from-zero-to-kubernetes-part-1
series: from-zero-to-kubernetes
codeMaxLines: 40
usePageBundles: true
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
[Part 1 - Build the VM template (this post)](/2024/07/from-zero-to-kubernetes-part-1)

This post is part of a series to deploying and configuring Kubernetes using the latest component versions on VMware Photon OS v5, as I write up additional parts they will be linked above and in the 'Posts in this series' sidebar.

# Introduction {#introduction.wp-block-heading}

As I use Kubernetes (k8s) more frequently, I needed to learn more about the latest features (more recent than those typically available from most package managers). I also wanted to see how hard it would be to build a minimal OS template image suitable for experimenting with k8s cluster configurations based on VMware Photon OS 5. This post (and the accompanying [github][1] repository) are a snapshot of my progress so far towards this.

My main goals for this project were to:
  
- Build on Photon OS 5.0 minimal installation as the base operating system
- Use an automated VM template for easy repeatable deployments
- Use latest Kubernetes v1.30.2 / Containerd v1.7.19 / runc v1.1.13 components (at time of writing)
- Make it reasonably easy/straightforward to update component versions over time
- Use kubeadm to bootstrap the cluster
- Use kube-vip for High Availability (HA) of the kubernetes control-plane
- Use kube-vip-cloud-controller to provide load balancing services to other applications
- Use calico for CNI networking
- Support persistent storage for deployed applications (PVC) on NFS storage
- Support deployment of helm charts

# Design

I wanted to keep the deployment size as minimal as possible while still being 'large' enough to be useful, I settled on a sizing of 3 control-plane and 3 worker nodes for kubernetes each with 2 vCPU cores, 8GB RAM and 16 GB local storage each, the number and sizing of nodes can be easily adjusted as required.

In addition I provisioned a NFS share to act as persistent storage. This entire deployment fits comfortably on a small form-factor PC which I've upgraded to 64 GB RAM and 1 TB NVMe drive The PC is running VMware ESXi as the hypervisor host, but this process could be easily followed using any hypervisor. Performance so far seems extremely good even though the PC only has a 4-core i5-6500T CPU.

## Network Design

I defined the following networks to be used for the deployment (using [RFC 1918][2] private IP ranges). The ranges were chosen to provide plenty of address space for my use case and to allow 2nd (and subsequent) clusters to live within a reasonably contained address space (This layout gives me the ability to create up to 32 complete clusters in a single /15 network subnet - 172.30.0.0/15 in my case).

| Function         | CIDR          | Start of range | End of range  | Addresses | Notes                                                  |
| ---------------- | ------------- | -------------- | ------------- | --------- | ------------------------------------------------------ |
| Kubernetes Nodes | 172.30.0.0/24 | 172.30.0.0     | 172.30.0.255  | 256       | Used for cluster host VMs and kube-vip VIP             |
| Load Balancer    | 172.30.1.0/24 | 172.30.1.0     | 172.30.1.255  | 256       | kube-vip IP pool for load-balanced exposed services    |
| reserved         | 172.30.2.0/23 | 172.30.2.0     | 172.30.3.255  | 512       | Reserved for any future requirements for this cluster |
| Services network | 172.30.4.0/22 | 172.30.4.0     | 172.30.7.255  | 1024      | Internal services network                              |
| Pod network      | 172.30.8.0/21 | 172.30.8.0     | 172.30.15.255 | 2048      | Internal pod (container) addresses                     |

## Host Design

Three control-plane nodes and three worker nodes will be deployed in the cluster as in the table below, also shown are the 'cluster' IP address for `kube-vip` and the NFS server used to provide persistent storage:

| Hostname    | IP Address  | Subnet mask       | Gateway    | Notes                                               |
| ----------- | ----------- | ----------------- | ---------- | --------------------------------------------------- |
| k8s01-nfs01 | 172.30.0.5  | 255.254.0.0 (/15) | 172.30.0.1 | NFS Server for persistent storage                   |
| k8s01       | 172.30.0.10 | 255.254.0.0 (/15) | 172.30.0.1 | 'Floating' HA IP address for kube-vip               |
| k8s01-c01   | 172.30.0.11 | 255.254.0.0 (/15) | 172.30.0.1 | Control plane node #1                               |
| k8s01-c02   | 172.30.0.12 | 255.254.0.0 (/15) | 172.30.0.1 | Control plane node #2                               |
| k8s01-c03   | 172.30.0.13 | 255.254.0.0 (/15) | 172.30.0.1 | Control plane node #3                               |
| k8s01-w01   | 172.30.0.14 | 255.254.0.0 (/15) | 172.30.0.1 | Worker node #1                                      |
| k8s01-w02   | 172.30.0.15 | 255.254.0.0 (/15) | 172.30.0.1 | Worker node #2                                      |
| k8s01-w03   | 172.30.0.16 | 255.254.0.0 (/15) | 172.30.0.1 | Worker node #3                                      |

I have a local DNS environment so just added all of the host details here to my primary DNS domain as 'A' records (including the `k8s01` and `k8s01-nfs01` names). The same effect can probably be achieved by creating `/etc/hosts` entries for each address on each deployed server (ideally in the template VM so these are automatically present).

# Automation

It quickly became apparent that the best way to obtain reproducible cluster deployments would be to automate the build of a template VM from which cluster VMs could be cloned and deployed. For those wishing to just use the automation scripting that I developed, a link to the github repository is [here][3]

The sections below detail the actions performed by the automation build, together with some of the reasons for choices made in the configurations.

## Template VM Build

For those wanting to build or customise the template image themselves, the steps are included below.

### Pre-requisites:

Build a new VM to act as the template for deploying the cluster - this VM will not be used directly but will be cloned to each of the require control-plane and worker nodes later. Specs are based on my experience and work fine - you will probably be able to get away with smaller sizing if CPU and/or RAM are limited:

- Install a new VM from photon 5 minimal ISO image (2 vCPU/8 GB vRAM/16 GB Disk using 'Thin' provisioning)
- Template name (e.g. k8s-tmpl) for hostname
- Use DHCP networking (for now) - use a network supporting DHCP clients that can reach the internet
- Set initial root password during install and complete initial Photon OS install / remove ISO attachment & reboot after install
- Login as root on console and check the DHCP IP address assigned (<code>ip a</code>)

### Enable SSH for root

Many of the steps below are easier to copy/paste to a terminal session rather than retyping on the VM console - to allow external SSH session (e.g. from your desktop), the following allows SSH connections to the `root` user.

{{% notice warning "Warning" %}}
Enabling ssh login for the `root` login is not good security practice (it's disabled by default for a reason). Don't do this on a public or untrusted network.
{{% /notice %}}

```bash
sed -i 's/PermitRootLogin no/PermitRootLogin yes/' /etc/ssh/sshd_config
systemctl restart sshd
```

You may find it easier at this point to ssh to the template VM and do the remainder of the configuration from a SSH client (e.g. puTTY or similar).

### Configure Firewall

Kubernetes and addon services need to be able to communicate between the deployed control-plane and worker nodes, the following ports are required (except icmp which I just include for convenience/testing connectivity):<figure class="wp-block-table is-style-stripes">

|Port|Protocol|Control-Plane Nodes|Worker Nodes|Description|
|:--:|:---:|:---:|:---:|---|
|179  |tcp|Yes|Yes|Calico BGP networking|
|2379 |tcp|No |Yes|etcd & kube-apiserver|
|2380 |tcp|No |Yes|etcd & kube-apiserver|
|4789 |udp|Yes|Yes|Calico networking with VXLAN enabled|
|5473 |tcp|Yes|Yes|Calico networking with Typha enabled|
|6443 |tcp|Yes|No |Kubernetes API server|
|10250|tcp|Yes|Yes|kubelet API server|
|10256|tcp|Yes|No |Kubernetes Load balancers|
|10257|tcp|No |Yes|kube-controller-manager|
|10259|tcp|No |Yes|kube-scheduler|
|30000-32767|tcp|Yes *|No *|NodePort Services - see note below|
|all|ip-in-ip|Yes|Yes|Calico IP-in-IP|
|all|icmp|No|No|Allows ping between cluster nodes|

{{% notice info "* Note on NodePort firewall rule" %}}
This is the default port range used by NodePort services (it can be changed) and if the control-plane servers are 'untainted' (to allow them to also schedule pod workloads) will also be required on control-plane nodes. In our deployment we're not planning on using NodePort services so this could safely be omitted from the firewall rules.
{{% /notice %}}

The rules created below cover both control-plane and worker node requirements (both sets are included to simplify template) and to save them to be persistent across host restarts:

```bash
iptables -A INPUT -p tcp --dport 179 -j ACCEPT
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
iptables-save > /etc/systemd/scripts/ip4save
```

### Configure ssh keys

{{% notice warning "Warning" %}}
Setting up ssh keys in the template in the way described here will allow all deployed VMs from the template to `ssh` (to each other) **without passwords**. Obviously this is a security risk and combined with enabling the ssh service for the `root` user shouldn't be used anywhere near a public or untrusted network.
{{% /notice %}}

Generate public/private key pair with no passphrase:

```bash
ssh-keygen -t rsa -b 4096 -N '' <<< $'\ny' >/dev/null 2>&1
```

Add public key to authorized_keys file (so that each deployed server fromm the template VM will trust the others and allow passwordless logins)

```bash
cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
```

### Configure kernel modules

Add the `overlay` and `br_netfilter` kernel modules to be loaded automatically (they will be loaded after the reboot after the next step):

```bash
echo overlay > /etc/modules-load.d/20-containerd.conf
echo br_netfilter >> /etc/modules-load.d/20-containerd.conf
```

### Package removes/adds/updates

The latest version of containerd in Photon OS installed by tdnf updates is from May 2023, remove and replace by current release (at time of writing) version and remove docker and associated files:

Requires internet connectivity for the template VM, reboot to ensure updated kernel & modules are loaded as well as pre-existing aliases for containerd are removed:

```bash
tdnf remove -y docker* containerd* runc
tdnf install -y wget tar jq socat ethtool conntrack git nfs-utils
tdnf update -y
reboot
```

Once the VM has restarted, reconnect via ssh as `root` and continue.

### Configure kernel settings

The pre-defined `50-security-hardening.conf` file disables ipv4 forwarding so make sure we are after that in the execution order by setting a higher numeric value on our configuration:

```bash
echo net.ipv4.ip_forward=1 > /etc/sysctl.d/60-kubernetes.conf
sysctl --system
```

### (Optional) Make a downloads folder and cd into it

This is not required, but this way all of the downloads can be easily cleaned up prior to turning the VM into a template to reduce overall storage

```bash
mkdir ~/k8s-downloads
cd ~/k8s-downloads
```

### Install containerd

Check [here][4] for latest release and copy link then install:

```bash
wget https://github.com/containerd/containerd/releases/download/v1.7.19/containerd-1.7.19-linux-amd64.tar.gz
tar Czxvf /usr/local containerd-1.7.19-linux-amd64.tar.gz
wget -P /usr/local/lib/systemd/system https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
systemctl daemon-reload
systemctl enable --now containerd
```

### Install updated runc

I'm not sure if this is necessary - but figured that it can't hurt to have latest release from [here][5] then install (this is definitely needed if the original runc was removed earlier):

```bash
wget https://github.com/opencontainers/runc/releases/download/v1.1.13/runc.amd64
install runc.amd64 -o root -g root -m 0755 /usr/bin/runc
```

### Install CNI plugins

Check [here][6] and grab latest CNI plugins package and install:

```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.5.1/cni-plugins-linux-amd64-v1.5.1.tgz
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.1.tgz
```

### Configure containerd

Generate default containerd configuration file, then edit to enable systemd Cgroups and use an updated `pause` image to prevent warnings from `kubeadm`:

```bash
/usr/local/bin/containerd config default > /etc/containerd/config.toml
sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/' /etc/containerd/config.toml
sed -i 's/pause\:3.8/pause\:3.9/' /etc/containerd/config.toml
```

### (Optional) Install nerdctl

Check releases [here][7] and grab 'latest' and install:

```bash
wget https://github.com/containerd/nerdctl/releases/download/v1.7.6/nerdctl-1.7.6-linux-amd64.tar.gz
tar Cxzvf /usr/local/bin nerdctl-1.7.6-linux-amd64.tar.gz
```

Provides nerdctl which can be useful to troubleshoot kubernetes containers:

e.g.
`nerdctl --namespace k8s.io ps -a`

### Install kubernetes binaries:

Pre-stage all of the binaries and service definitions in the template so they're ready to go:

```bash
# kubectl
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
```

### Setup bash aliases & completion

This setups up bash completion for `kubectl` and also allows `k` to be used (instead of `kubectl`) as an alias:

```bash
cat <<EOF >> $HOME/.profile
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
```

### Tidy up and prepare template for cloning

We can now remove all of the downloaded files and ensure that ssh keys and `/etc/machine-id` are recreated when the template is cloned and the resulting VM powered on for the first time. Finally we shutdown the template VM ready for capture/cloning:

```bash
cd ~
rm -rf ~/k8s-downloads
rm -rf /etc/ssh/*_key
rm -rf /etc/ssh/*pub
echo "" > /etc/machine-id
shutdown -h now
```

Now convert the template VM to a template (or capture to a content library) ready to be cloned/deployed to build our actual cluster VMs.

As always, comments & feedback welcome.

Jon.

 [1]: https://github.com/jondwaite/k8s-photon
 [2]: https://datatracker.ietf.org/doc/html/rfc1918
 [3]: https://github.com/jondwaite/k8s-photon/
 [4]: https://github.com/containerd/containerd/releases
 [5]: https://github.com/opencontainers/runc/releases
 [6]: https://github.com/containernetworking/plugins/releases
 [7]: https://github.com/containerd/nerdctl/releases/