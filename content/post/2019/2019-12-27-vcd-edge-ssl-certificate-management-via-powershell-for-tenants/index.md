---
title: vCD Edge SSL Certificate Management via PowerShell for tenants
author: Jon Waite
type: post
date: 2019-12-27T00:10:59+00:00
url: /2019/12/vcd-edge-ssl-certificate-management-via-powershell-for-tenants/
categories:
  - Networking
  - NSX
  - PowerCLI
  - PowerShell
  - REST API
  - vCloud Director
tags:
  - NSX
  - PowerCLI
  - PowerShell
  - SSL
  - vCloud Director

---
Tom Fojta has a couple of really good blog posts on his blog (<https://fojta.wordpress.com/>) about using Let's Encrypt certificates on an NSX Edge Load Balancer. The first part can be found at <https://fojta.wordpress.com/2016/07/16/automate-lets-encrypt-certificate-for-nsx-edge-load-balancer/> and the second part at <https://fojta.wordpress.com/2019/12/21/automate-lets-encrypt-certificates-part-2/>. The method described in these posts relies on connectivity with the NSX management API, which is fine in an enterprise environment, but won't work for tenants of a Service Provider vCloud Director environment where the NSX API is not directly accessible but needs to be accessed via a vCloud Director proxy.

VMware document the NSX proxy functionality [here][1] which allows vCloud API clients to make requests to the NSX API.

So I decided to see if I could get a script working that performed the same 'certificate refresh' in Tom's posts working as a tenant in a vCloud Director environment as a tenant, the basic functionality I wanted was to:

  * Upload a new/replacement SSL certificate to the NSX Edge.
  * Change a Load Balancer application profile to use the new SSL certificate.
  * (Optionally) remove the 'old' certificate from the NSX Edge.

Ideally this would all be easily scriptable so that it could be triggered regularly (e.g. monthly) to continually extend the certificate life when using short-duration certificates such as those from Let's Encrypt.

I found it was reasonably straightforward to get the script working via the vCloud proxy API to NSX, so I extended it a bit and turned it into a proper PowerShell module which I've also published to PS Gallery.

The project is hosted on my Github at this link: <https://github.com/jondwaite/vCDEdgeSSL>. I've also included a README.md with full documentation of the cmdlets made available from the module and examples of how these can be used.

There are still a couple of issues with the module which means it won't currently work on PowerShell Core, but I hope to get these fixed and a new version uploaded which fixes this - will update this post once done.

As always, appreciate any comments/feedback, I think this module could be great for anyone wanting to use short-duration SSL certificates on services published via an NSX Edge Gateway as a tenant in a service provider environment.

Jon

 [1]: https://code.vmware.com/docs/6900/vcloud-director-api-for-nsx-programming-guide