---
title: Configuring Network Multipathing for TrueNAS Scale and ESXi
author: Jon Waite
type: post
date: 2024-06-05T09:20:27+00:00
url: /2024/06/configuring-network-multipathing-for-truenas-scale-and-esxi/
codeMaxLines: 15
categories:
  - Networking
  - TrueNAS
  - VMware
tags:
  - Multipathing
  - Network
  - NMP
  - SATP
  - TrueNAS Scale
  - VMware

---
Recently I've been testing out various NAS configurations in my homelab environment - I was particularly keen to get [TrueNAS Scale][1] (TNS) working with physical and virtual VMware ESXi hosts.

I'd used NFS before from TNS to ESXi servers but found performance somewhat lacking (even when using NVMe or SSD storage) and multipathing with NFS4, so for this attempt I decided to give iSCSI a try.

I'd seen some forum posts indicating that Asymmetric Logical Unit Access (ALUA) was supported by TNS and the Storage Array Type Plugin (SATP) VMW\_SATP\_ALUA should be used to configure ESXi multipathing claim rules with a command such as the following used on the ESXi host:

`esxcli storage nmp satp rule add -s "VMW_SATP_ALUA" -V "TrueNAS" -M "iSCSI Disk" -P "VMW_PSP_RR" -O "policy=iops;iops=1;" -e "TrueNAS iSCSI Claim Rule"`

Note: The advantage of configuring SATP claim rules in this way is that any presented block storage which matches the 'Vendor' (-V option) and 'Model' (-M option) specified in the rule will automatically have the multipathing settings applied which avoids having to manually configure these for each block device.

Howvere, when I tried this configuration in my homelab, I found that ESXi could no longer see any presented iSCSI devices from the TNS system and the datastores I'd created on the TNS storage became inaccessible. (Note that changes to SATP rules are typically only evaluated at system boot so after creating the rule I restarted the server to be sure, but the storage remained invisible).

Checking the ESXi logs showed many errors similar to these (I've put the text of the errors here in the hope that Google and other search engines will find them if people search on these error messages):

```log
WARNING: NMP: nmp_PspSet:527: Switching to claimrule PSP "VMW_PSP_RR" for device Unregistered Device.
WARNING: NMP: nmp_SatpClaimPath:2193: SATP "VMW_SATP_ALUA" could not add path "vmhba65:C0:T0:L1" for device "Unregistered Device". Error Not supported
WARNING: NMP: nmp_DeviceAlloc:2185: nmp_AddPathToDevice failed Not supported (195887136).
WARNING: NMP: nmp_DeviceAlloc:2205: Could not allocate NMP device.
WARNING: ScsiPath: 8621: Plugin 'NMP' had an error (Not supported) while claiming path 'vmhba65:C0:T0:L1'. Skipping the path.
```

It appeared something was not working with ALUA negotiation between ESX and TNS, to confirm the issue I tried the same from both the latest patched versions of ESXi 7 and ESXi 8 (at the time of writing in June 2024) with the same results. Further research indicated that ALUA is only supported on the TrueNAS Enterprise products rather than the non-Enterprise edition I was using in my homelab.

To confirm this, I removed the SATP rule and replaced it with a new rule using the `VMW_SATP_DEFAULT_AA` configuration as follows:

First to remove the incorrect rule:

`esxcli storage nmp satp rule remove -s "VMW_SATP_ALUA" -V "TrueNAS" -M "iSCSI Disk" -P "VMW_PSP_RR" -O "policy=iops;iops=1;" -e "TrueNAS iSCSI Claim Rule"`

The to add back the 'fixed' rule:

`esxcli storage nmp satp rule add -s "VMW_SATP_DEFAULT_AA" -V "TrueNAS" -M "iSCSI Disk" -P "VMW_PSP_RR" -O "policy=iops;iops=1;" -e "TrueNAS iSCSI Claim Rule"`

After restarting the host, the log errors did not reappear and the TNS devices and datastores were once again available and accessible. In addition, checking the multipathing information before any rules and after applying the 'fixed' rule confirmed that all 4 network paths in my homelab setup were now active:

Before view (prior to either ALUA or DEFAULT_AA SATP rules):

```bash
esxcli storage nmp device list -d naa.6589cfc00000027e04e9c9a5e50c0fe5
naa.6589cfc00000027e04e9c9a5e50c0fe5
Device Display Name: TrueNAS iSCSI Disk (naa.6589cfc00000027e04e9c9a5e50c0fe5)
Storage Array Type: VMW_SATP_DEFAULT_AA
Storage Array Type Device Config: {action_OnRetryErrors=off}
Path Selection Policy: VMW_PSP_FIXED
Path Selection Policy Device Config: {preferred=vmhba65:C3:T0:L1;current=vmhba65:C3:T0:L1}
Path Selection Policy Device Custom Config:
Working Paths: vmhba65:C3:T0:L1
Is USB: false
```

After applying DEFAULT_AA SATP rule and restarting host:

```bash {hl_lines=[6,7,9]}
esxcli storage nmp device list -d naa.6589cfc00000027e04e9c9a5e50c0fe5
naa.6589cfc00000027e04e9c9a5e50c0fe5
Device Display Name: TrueNAS iSCSI Disk (naa.6589cfc00000027e04e9c9a5e50c0fe5)
Storage Array Type: VMW_SATP_DEFAULT_AA
Storage Array Type Device Config: {action_OnRetryErrors=off}
Path Selection Policy: VMW_PSP_RR
Path Selection Policy Device Config: {policy=iops,iops=1,bytes=10485760,useANO=0; lastPathIndex=3: NumIOsPending=0,numBytesPending=0}
Path Selection Policy Device Custom Config:
Working Paths: vmhba65:C3:T0:L1, vmhba65:C2:T0:L1, vmhba65:C1:T0:L1, vmhba65:C0:T0:L1
Is USB: false
```

I'll definitely be doing some more testing, and writing up additional posts with the full end-to-end configurations and scripts I used to simplify configuring the hosts for iSCSI with TNS which hopefully will be of use to others when I get time. Initial indications are that all is now working correctly with the new claim rule and I'm getting a performance boost over using a single network link to the storage.

As always, comments and feedback appreciated, hope this post is useful to someone else encountering the same issue.

Jon.

 [1]: https://www.truenas.com/truenas-scale/