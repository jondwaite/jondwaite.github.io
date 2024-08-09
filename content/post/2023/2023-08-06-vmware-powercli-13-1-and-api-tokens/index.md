---
title: VMware PowerCLI 13.1 and API Tokens
author: Jon Waite
type: post
date: 2023-08-05T22:37:11+00:00
url: /2023/08/vmware-powercli-13-1-and-api-tokens/
categories:
  - Cloud Director
  - PowerCLI
  - Uncategorized
tags:
  - PowerCLI
  - REST API
  - vCloud Director
  - VMware

---
I previously wrote [here](/2021/10/how-to-use-api-tokens-in-vmware-cloud-director-10-3-1/) and [here](/2021/10/using-api-tokens-for-vcd-10-3-1-from-powercli-and-terraform/) about using the new VMware Cloud Director (VCD) API tokens functionality first introduced in version 10.3.1 to allow scripts and automation to connect to a VCD environment. With the update to VMware PowerCLI version 13.1 the previous way of achieving this is deprecated as the x-vcloud-authorization header is no longer used as VMware continue the move to cloudapi and now instead we can simply use a bearer token.

This actually simplifies connecting to VCD using PowerCLI with an API token, but means that the script I previously published [on github](https://github.com/jondwaite/vcdapitoken) won't work with PowerCLI versions 13.1 (or later). I've now renamed the `vcd-token.ps1` script in github to `vcd-token-legacy.ps1` which should only be used on PowerCLI versions 13.0 and prior, for PowerCLI version 13.1 and later the replacement `vcd-token.ps1` should now be used.

I've also added a script option to skip verification of SSL certificate checks which can be useful while testing in non-production environments.

If you just want to use this in your scripting or automation workflows then simply grab [this](https://raw.githubusercontent.com/jondwaite/vcdapitoken/main/vcd-token.ps1) code and include/adjust it as necessary for your environment.

I've tested this using PowerCLI 13.1, PowerShell Core 7.3.6 against a test VCD environment running the recently released version 10.5 where it all works fine, but any version of PowerCLI 13.1+ and VCD 10.3.1+ should also be fine.

As always, any comments/feedback welcome, and if you have issues please feel free to raise an issue against my [github repository](https://github.com/jondwaite/vcdapitoken).

Jon