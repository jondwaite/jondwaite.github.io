---
title: First thoughts on AWS Re:Invent 2018 and where VMware is missing the point
author: Jon Waite
type: post
date: 2018-11-30T01:46:14+00:00
url: /2018/11/first-thoughts-on-aws-reinvent-2018-and-where-vmware-is-missing-the-point/
categories:
  - AWS
  - vCloud Director
  - VMware

---
Firstly I’ll start this off by saying that I don’t usually write opinion pieces – I’d much rather share some cool technology or tips that I’ve come across in my day job (or even playing with technology in my own time). Secondly, my perspective is likely a bit skewed – I work in one of the most virtualized and cloud-adopting countries in the world (New Zealand) where the vast majority of customers I speak to have no on-premises server environment any more or are planning to retire the ones they have. These customers have either shifted entirely to public cloud, or are using the services of providers such as my employer to provide local private cloud platforms for them.

In particular in Christchurch where I’m based we had a series of devastating earthquakes in 2010 and 2011 which accelerated this shift – many customers simply felt no need to rebuild datacenters when rebuilding their offices given available local provider and public cloud options open to them. Having a high-speed urban fibre network across the entire country has almost certainly helped to accelerate this trend.

For those not familiar, hosting providers such as ourselves who run a largely VMware software stack use the usual components (vSphere, vCenter, ESXi, NSX networking and sometimes VSAN storage) as the foundation of our platforms, but VMware provide an additional software layer ‘vCloud Director’ (vCD) which sits on top of all of these to provide a secure multi-tenant platform. It also provides a feature-rich public-facing portal and API to allow orchestration and automation as well as being extensible by a plugin architecture so that Operations Management, Backup/Recovery, Replication, Container hosting (to name 4) and other services can be easily integrated and published to our customers. The way this architecture has been implemented also makes it reasonably easy for 3rd parties to write their own vCD extensions and have these seamlessly published into the same environment.

Recent improvements in vCD 9.5 have included the move to a native HTML5 portal, the addition of many more customization and configuration options, support for multi-site deployments where customers consume resources in multiple physical locations, as well as new networking functionality allowing seamless networking across multi-site deployments. In addition, the new tenant portals for vCloud Availability Cloud to Cloud DR and vRealize Operations allow customers to completely manage their DR replication and failover as well as gain operational insights and management of their deployed workloads. <a href="https://blogs.vmware.com/vcloud/2018/11/vmware-vcloud-director-9-5-the-new-features-in-detail.html" target="_blank" rel="noopener" class="broken_link">This blog post</a> covers the most recently enabled functionality in vCloud Director for those that wish to find more information.

Many of our customers are also leveraging public cloud infrastructure platforms for a variety of reasons including advanced features, hyper-scale elasticity, ease of operations and management and (often) to decrease their overall IT infrastructure spend. Often overlooked though is that the main driver for many customers is to have their internal IT teams concentrating on business applications and data - looking for ways to add business value to their organizations rather than in ‘feeding and watering’ infrastructure in their own datacenters. In fact many of the reasons customers chose a provider such as ourselves are very similar. The determining factors are often older applications that can’t survive at the reasonably significant latencies which are inevitable from New Zealand to our closest public cloud platforms in Australia, and some concerns around data sovereignty (although these are largely diminishing).

Of the AWS announcements made this week at their annual Re:Invent , AWS Outposts is a fascinating platform proposition which, if done well, will be a great option for a local AWS consistent platform in local (NZ) datacenters. I’m also  impressed with the announcements for new and enhanced AWS services - S3 Glacier Deep Archive could well spell the ‘final’ end of tape as a data archival technology for example. AWS Control Tower as a simplified way to easily deploy a landing zone into AWS is another service which I think will resonate well with many of our customers who find it challenging to deploy their initial AWS footprints with appropriate security, controls and governance. AWS TimeStream finally provides a ‘proper’ way of dealing with time-series data in huge volumes without trying to squish it into a relational database with all the issues that creates. Perhaps the most interesting is AWS RDS on VMware which was announced back in August at VMworld and allows AWS RDS services to run in a vSphere environment in a local datacenter with support for data replication and DR.

So with all that said, why do I think VMware is missing the point? – after all there are some great technologies and services being made available from both vendors.

Let’s take a look at some new/recent VMware products and services and see:

1) <a href="https://blogs.vmware.com/management/2018/08/introducing-cloud-automation.html" target="_blank" rel="noopener">VMware Cloud Assembly</a>

A great new technology to allow easy construction of templates and blueprints to speed deployment of application environments to multiple cloud endpoints. Supports all the major public cloud endpoints (AWS, Azure, GCP) as well as vCenter as the deployment endpoint.

2) <a href="https://cloud.vmware.com/vmware-hcx" target="_blank" rel="noopener">VMware HCX (Hybrid Cloud Extension)</a>

Awesome technology which allows live-migration of running business applications between vSphere sites (and even between vSphere on-premises and VMware cloud on AWS).

3) <a href="https://aws.amazon.com/rds/vmware/" target="_blank" rel="noopener">AWS RDS on vSphere</a>

Mentioned above, but provides capability to run AWS consistent database services from all major RDS providers (MariaDB, MySQL, PostgreSQL, Oracle and SQL Server) in an on-premises vSphere environment. Can even allow these databases to span both an on-premises and AWS environment to provide scale, high availability and DR options.

4) <a href="https://cloud.vmware.com/vmc-aws" target="_blank" rel="noopener">VMware Cloud on AWS</a>

Fantastic option that gives customers the option of deploying vSphere environments directly into public cloud and run their VMs with no changes whatsoever in that platform. Also with HCX (above) can live-migrate workloads in and out of public cloud. Provides consistent management, operations and security options across both platforms.

What do all of these have in common? You’ve probably guessed it – not a single one of them works with vCloud Director. If a customer wants to use HCX (for example) to seamlessly move workloads from a VCPP provider to AWS and back – not supported. Deploy Cloud Assembly blueprints to their VCPP provider? Nope, doesn’t work either, no support for the vCD API as an endpoint. Allow them to use AWS RDS services alongside their VCPP provider hosted VMs? No there also.

Basically it comes down to this: If you build a tool or service that only talks to vCenter (or vCenter APIs) and not to vCloud Director, you are missing out on making your products and services available to a large number (around 4,200 I believe right now) of VMware Cloud Provider Partners (VCPP) such as ourselves that offer vCloud Director as the primary interface and API for customers to manage their workloads. What’s more, from figures mentioned by VMware themselves, the number of workloads in VCPP provider datacenters managed through vCD is increasing massively ahead of vSphere and vCenter on-premises solutions.

One of the likely comments I’ll get to this post is ‘Well, you could just provided dedicated vSphere environments for each customer that needs these functions’. This is accurate – we definitely could do this, but the overhead of managing and maintaining a large number of discreet vSphere instances (including all of the management and operations tooling that these require) doesn’t scale well and would result in a huge amount of extra work. In addition, because we can’t securely multi-tenant vSphere environments there would be a huge amount of wasted capacity on hosts which aren’t heavily loaded or only exist to provide cluster hardware redundancy. This would make the solutions incredibly expensive by comparison to a true multi-tenanted platform.

So… if anyone from VMware is still reading by now… you’ve given your VCPP partners and providers an awesome platform in vCloud Director to allow a true multi-tenanted cloud platform, uptake and usage of this platform is in massive growth right now. Now please make sure the rest of your technologies can work with it.

As always, comments and feedback appreciated, how do other VCPP providers feel about this?

Jon

**Update:**

A few days after I first posted this, I saw [this tweet][1] from Steve Dockar pop up in my news feed linking to [this youtube video][2] which shows a preview version of Cloud Assembly using vCloud Director as the endpoint. This is awesome, and something which the VMware people I spoke to on the show floor at re:Invent obviously knew nothing about.

 [1]: https://twitter.com/SteveDockar/status/1070021374696136704
 [2]: https://www.youtube.com/watch?v=gJeJk_8kjzg