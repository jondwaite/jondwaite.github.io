---
title: vCloud Availability 3.0 – Working with the vCAV API
author: Jon Waite
type: post
date: 2019-05-11T10:23:46+00:00
url: /2019/05/vcloud-availability-3-0-working-with-the-vcav-api/
categories:
  - vCloud Availability
  - vCloud Director
  - VMware

---
One of the most welcome additions in vCloud Availability (vCAV) 3.0 is a public API which exposes much of the platform capability to automation and orchestration. In particular for Service Providers it is possible to relatively easily extract statistics on the number of replicated VMs, the type of replication (ongoing protection or one-off migrations), the storage consumption occupied by Point-In-Time instances and replication status.

It is also possible (I have yet to test this) to enable full configuration and re-configuration of replication via the vCAV API and this is something definitely on my list to test further.

VMware provide 2 python scripts in the vCAV appliances (usage-report.py and storage-report.py) which provide some good base information, but having to login to the appliances and run these locally under the vCAV appliance root user account isn't ideal -&nbsp;as a Service Provider I'd like to be able to remotely interrogate the API and retrieve information on configured replications for billing and service monitoring purposes.

VMware has published the vCAV public API specification at&nbsp;<https://code.vmware.com/apis/441/vcav> but at this stage I'm unsure of the exact status of this - some conversations with VMware staff have indicated that this is not 'officially released or supported'. Undeterred I decided to see what could be done to consume this API and write a small PowerShell module to make it easier to consume the API using vCloud Director session credentials rather than relying on 'root' user access to the appliances themselves.

**Note:** I have definitely noticed some inconsistencies between the API usage in the VMware scripts and what is currently documented on code.vmware.com. In some cases this prevented API calls from working and I had to reverse-engineer the calls from c4cli.py from the appliances to get the correct syntax. This may also explain why the public vCAV API is not yet officially supported.

The results of my experimentation and development have been published to github here:&nbsp;<https://github.com/jondwaite/PowerVCAV> and I've also made the PowerVCAV module available in PowerShell Gallery so that it can be easily installed using the PowerShell Install-Module cmdlet. Note that PowerVCAV relies on connection information from PowerCLI and the Connect-CIServer cmdlet so this is required.

PowerVCAV consists of 6 cmdlets to assist in managing vCAV session connections and allow easy querying and consumption of the vCAV API.

I've included documentation for each of the cmdlets in the github repo readme, together with some examples of the connection process and syntax.

Hopefully this module will prove useful to others who need to work with the vCAV API and provide a foundation for being able to build queries against this.

As always, comments and feedback appreciated, and if you have any suggestions for improvements feel free to log a request against the github repo.

Jon