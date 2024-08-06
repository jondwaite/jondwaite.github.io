---
title: Using API Tokens for VCD 10.3.1 from PowerCLI and Terraform
author: Jon Waite
type: post
date: 2021-10-22T08:35:29+00:00
url: /2021/10/using-api-tokens-for-vcd-10-3-1-from-powercli-and-terraform/
categories:
  - Cloud Director
  - PowerShell
  - REST API
tags:
  - Bash
  - PowerCLI
  - PowerShell
  - REST API
  - Terraform
  - vCloud Director
  - VMware

---
I had some queries on my [previous blog post][1] on using the new API Tokens in VMware Cloud Director (VCD) 10.3.1 about whether these can be used in PowerCLI scripts from PowerShell or Terraform scripts which use the VMware Terraform Provider against Cloud Director. Fortunately the answer to both questions is 'Yes' and I've created a [small github repository][2] with examples of how these can be done.

For both scripts the variables need to be set to something appropriate:  
`org` should be set to the 'short' organization code for VCD (the same name as you'd use after 'tenant' when accessing the VCD UI)  
`vcdhost` should be set to the FQDN of the VCD environment you are using (e.g. cloud.my.service.provider.net)  
`token` should be set to the API Token value created in VCD

For PowerCLI we can use the `SessionId` returned from first refreshing the API token and then accessing the `/api/session` path to login with `Connect-CIServer` which provides full access to all PowerCLI cmdlets, the script to do this is included below and in the [Github repository][2] as `vcd-token.ps1`.

<pre class="EnlighterJSRAW" data-enlighter-language="powershell">$org = "<VCD Organization Code>"
$vcdhost = "<VCD Hostname>"
$token = "<VCD API Token String>"

# Use the token to generate an access-token
try {
  $uri = "https://$($vcdhost)/oauth/tenant/$($org)/token?grant_type=refresh_token&refresh_token=$($token)"
  $access_token = (Invoke-RestMethod -Method Post -Uri $uri -Headers @{'Accept' = 'application/json'}).access_token
  Write-Host -ForegroundColor Green ("Created access_token from token successfully")
} catch {
  Write-Host -ForegroundColor Red ("Could not create access_token from token, response code: $($_.Exception.Response.StatusCode.value__)")
  Write-Host -ForegroundColor Red ("Status Description: $($_.Exception.Response.ReasonPhrase).")
}

# Use the access-token to get a SessionId from the x-vcloud-authorization header response:
$headers = @{"Accept" = "application/*+xml;version=36.1"; "Authorization" = "Bearer $($access_token)"}
try {
  $SessionId = [string](Invoke-WebRequest -Method Get -Uri "https://$($vcdhost)/api/session" -Headers $headers).headers.'x-vcloud-authorization'
  Write-Host -ForegroundColor Green ("Got SessionId from access_token successfully")
} catch {
  Write-Host -ForegroundColor Red ("Could not create SessionId from access_token, response code: $($_.Exception.Response.StatusCode.value__)")
  Write-Host -ForegroundColor Red ("Status Description: $($_.Exception.Response.ReasonPhrase).")
}

# Create a new PowerCLI connection using the returned SessionId:
Connect-CIServer -Server $vcdhost -SessionId $SessionId

# Example PowerCLI to return all VMs in Org:
Get-CIVM | Format-Table

# (Optional) Disconnect the CIServer session:
Disconnect-CIServer -Server $vcdhost -Confirm:$false
</pre>

Usage should be pretty straightforward - just change the variables at the top of the file for the VCD Organization code, VCD API hostname and the Token generated in the VCD UI and the commands at the bottom of the file as appropriate.

For Terraform, we can use a small Bash script in a similar way to set a `VCD_TOKEN` environment variable which can then be consumed by the [VCD Terraform Provider][3] to authenticate when applying Terraform plans. This script is in the [Github repository][2] as `vcd-token.sh`.

<pre class="EnlighterJSRAW" data-enlighter-language="shell">#!/bin/bash

org="<VCD Organization Code>"
vcdhost="<VCD Hostname>"
token="<VCD API Token String>"

uri="https://$vcdhost/oauth/tenant/$org/token?grant_type=refresh_token&refresh_token=$token"
accesstok=`curl -s -k -X POST $uri -H "Accept: application/json" | jq -r '.access_token'`
headers=`curl -s -H "Accept: application/*+xml;version=36.1" -H "Authorization: Bearer $accesstok" -k -I -X GET https://$vcdhost/api/session`
export VCD_TOKEN=`echo "$headers" | grep X-VMWARE-VCLOUD-ACCESS-TOKEN: | cut -f2- -d: | awk '{$1=$1};1'`</pre>

The easiest way to use this is to save as a shell script (and make executable) and then 'dot source' this `. ./vcd-token.sh` or `source ./vcd-token.sh` prior to applying any Terraform plan that requires the token to be set.

Note that this script uses [jq][4] to parse the initial returned JSON so this will need to be available in the environment too.

Hopefully will be useful to those of you looking to use the new VCD 10.3.1 API token system to replace stored credentials in your automation scripting.

Jon.

 [1]: https://kiwicloud.ninja/?p=68945
 [2]: https://github.com/jondwaite/vcdapitoken
 [3]: https://registry.terraform.io/providers/vmware/vcd/latest/docs
 [4]: https://stedolan.github.io/jq/