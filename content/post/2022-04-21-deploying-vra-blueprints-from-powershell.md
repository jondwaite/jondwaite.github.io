---
title: Deploying vRA Blueprints from PowerShell
author: Jon Waite
type: post
date: 2022-04-21T11:09:47+00:00
url: /2022/04/deploying-vra-blueprints-from-powershell/
categories:
  - PowerShell
  - REST API
  - VMware
  - vRealize Automation
tags:
  - Automation
  - Blueprint
  - PowerShell
  - REST API
  - VMware
  - vRA

---
The [PowervRA][1] module is a great set of PowerShell cmdlets for managing a VMware vRealize Automation (vRA) environment, but I wanted to learn a bit more about the REST API available from vRA and also had a requirement &#8211; to script a deployment from an existing vRA blueprint (&#8216;Cloud Template&#8217;) which had no PowervRA cmdlet. So I set about writing my own script in PowerShell to do this and have documented this and the steps required in this post.

The steps in this process are:

<ul class="wp-block-list">
  <li>
    <a href="#gather-params">Gather the parameters and inputs for the deployment</a>
  </li>
  <li>
    <a href="#auth-1">Authenticate to the vRA API and obtain a refresh token</a>
  </li>
  <li>
    <a href="#auth-2">Use the obtained refresh token to obtain an access token</a>
  </li>
  <li>
    <a href="#find-project">Find the Project Id and Blueprint Id that we want to deploy from the API</a>
  </li>
  <li>
    <a href="#deploy">Request the deployment and check the request is submitted successfully</a>
  </li>
</ul>

I also wanted to provide some basic error checking so that if the specified vRA Project or Blueprint could not be found the script provided meaningful feedback.

### Gather the parameters and inputs for the deployment {#gather-params.has-medium-font-size.wp-block-heading}

In the first code block, I simply configure variables for the environment &#8211; including prompting for the password which will be used to authenticate to vRA:

<div class="wp-block-syntaxhighlighter-code ">
  <pre class="brush: powershell; auto-links: false; gutter: false; title: ; quick-code: false; notranslate" title="">
# Target vRealize Automation System
$vrauri = 'https://&lt;vRealize Automation URL e.g. https://myvraserver.mydomain.local&gt;'
$user = '&lt;user name to authenticate to vRA&gt;'
$pass = (Get-Credential -UserName $user).GetNetworkCredential().password
$header = @{'Content-Type'='application/json'}
</pre>
</div>

The next block defines the parameters for the blueprint that will be deployed, including which vRA Project will be the deployment target, which blueprint will be used and the input parameters required for deploying this instance. The blueprint I&#8217;m using is a 3-tier application that I&#8217;ve been working on to provide a more realistic demonstration of vRA capabilities. Obviously these parameters will vary depending on the environment and template being deployed

<div class="wp-block-syntaxhighlighter-code ">
  <pre class="brush: powershell; auto-links: false; gutter: false; title: ; quick-code: false; notranslate" title="">
# Blueprint Deploy Parameters:
$environment        = 'Production'
$blueprintName      = '3-Tier App'
$deploymentName     = 'MyDeployment'
$description        = 'Deploying a vRA blueprint from code'
$appliancepassword  = 'MySecretPassword'
$department         = 'it'
$db_flavor          = 'medium'
$app_flavor         = 'small'
$web_flavor         = 'small'
</pre>
</div>

### Authenticate to the vRA API and obtain a refresh token {#auth-1.has-medium-font-size.wp-block-heading}

Next we need to POST the authentication data to the `/csp/gateway/am/api/login?access_token` URL to obtain a refresh token:

<div class="wp-block-syntaxhighlighter-code ">
  <pre class="brush: powershell; auto-links: false; gutter: false; title: ; quick-code: false; notranslate" title="">
try {   # Get an API refresh token
    $refres = Invoke-RestMethod -Method Post `
        -Uri "$vrauri/csp/gateway/am/api/login?access_token" `
        -Headers $header `
        -Body (@{'username'=$user;'password'=$pass} |
        ConvertTo-Json)
} catch { 
    Write-Host -ForegroundColor Red ("Error status: $_.Exception.Response.StatusCode.value__")
    Exit
}
</pre>
</div>

I&#8217;ve included some basic error handling here so that if an error occurs (e.g. an incorrect user name or password is provided) this is caught and the script gracefully exits. ConvertTo-Json is used to ensure our parameters are correctly provided to the API.

### Use the obtained refresh token to obtain an access token {#auth-2.has-medium-font-size.wp-block-heading}

Once we have our refresh token, we can call the `/iaas/api/login` URL to obtain the bearer token which will be used to authenticate all our subsequent API requests.

<div class="wp-block-syntaxhighlighter-code ">
  <pre class="brush: powershell; auto-links: false; gutter: false; title: ; quick-code: false; notranslate" title="">
try {   # Get an API access token
    $access_token = Invoke-RestMethod -Method Post `
        -Uri "$vrauri/iaas/api/login" `
        -Headers $header `
        -Body (@{'refreshToken'=$refres.refresh_token} | ConvertTo-Json)
} catch { 
    Write-Host -ForegroundColor Red ("Error status: $_.Exception.Response.StatusCode.value__")
    Exit 
}
</pre>
</div>

### Find the Project Id and Blueprint Id that we want to deploy from the API {#find-project.has-medium-font-size.wp-block-heading}

If everything has gone ok up to this point, we now have a token (in `$access_token.token`) which can be used to authenticate our next requests. In order to be able to request deployment of our blueprint we need to provide both the Project Id and Blueprint Id to the API. As I didn&#8217;t want to have to keep looking up and coding these Id&#8217;s into scripts I&#8217;ve used 2 calls to search and match these by name (`$environment` is the &#8216;friendly&#8217; Project name and `$blueprintName` is the &#8216;friendly&#8217; blueprint name in this example). Since both API calls use the same headers, we can set up a single `$header` variable and use it for both calls containing our authorization token.

<div class="wp-block-syntaxhighlighter-code ">
  <pre class="brush: powershell; auto-links: false; gutter: false; title: ; quick-code: false; notranslate" title="">
# Get the projectid for the deployment environment
$header = @{'Authorization'='Bearer ' + $access_token.token;'Content-Type'='*/*'}
$projects = (Invoke-RestMethod -Method Get `
    -Uri "$vrauri/project-service/api/projects" `
    -Headers $header).content
$projectid = ($projects | Where-Object { $_.name -eq $environment }).id
if (!$projectid) { 
    Write-Host -ForegroundColor Red "Environment $environment could not be located, exiting."
    Exit 
}

# Get the blueprintid for requested blueprint
$blueprints = (Invoke-RestMethod -Method Get `
    -Uri "$vrauri/blueprint/api/blueprints" `
    -Headers $header).content
$blueprintid = ($blueprints | Where-Object {$_.name -eq $blueprintName}).id
if (!$blueprintid) { 
    Write-Host -ForegroundColor Red "Blueprint $blueprintName could not be located, existing."
    Exit
}
</pre>
</div>

### Request the deployment and check the request is submitted successfully {#deploy.has-medium-font-size.wp-block-heading}

Now we have all of the information required, we can build a JSON object containing all our inputs to the deployment process and then make an API POST request to /blueprint/api/blueprint-requests to initiate the actual deployment.

<div class="wp-block-syntaxhighlighter-code ">
  <pre class="brush: powershell; gutter: false; title: ; notranslate" title="">
# Create 'body' for API deployment request
$deploybody = @{
    'blueprintId'       =$blueprintid # found from API
    'deploymentName'    =$deploymentName
    'description'       =$description
    'projectId'         =$projectid # found from API
    'inputs'=@{
        'password'      =$appliancepassword
        'department'    =$department
        'db_flavor'     =$db_flavor
        'app_flavor'    =$app_flavor
        'web_flavor'    =$web_flavor
    }
}
$header = @{'Authorization'='Bearer ' + $access_token.token;'Content-Type'='application/json'}

try {   # Deploy the blueprint
    $res = Invoke-RestMethod -Method Post `
         -Uri "$vrauri/blueprint/api/blueprint-requests" `
         -Headers $header `
         -Body ($deploybody | ConvertTo-Json)
} catch {
    Write-Host -ForegroundColor Red ("Error status: $_.Exception.Response.StatusCode.value__")
    Exit 
}

if ($res.status -eq 'CREATED' -or $res.status -eq 'STARTED') {
    Write-Host ("Deployment request successfully submitted, deploymentId is $($res.deploymentId).")
} else {
    Write-Host -ForegroundColor Red ("Deployment request failed, returned data shown below.")
    $res
}
</pre>
</div>

### Summary {.has-medium-font-size.wp-block-heading}

Hopefully this post is detailed enough that it will be easy to adjust for your particular blueprints and requirements if needed. I&#8217;ve included the entire script in one block below for easier copying/pasting if you want to try this in your own environment.

As always, comments, feedback and suggestions for improvement are welcome.

Jon

<div class="wp-block-syntaxhighlighter-code ">
  <pre class="brush: powershell; gutter: false; title: ; notranslate" title="">
# Complete code listing - vRA automation to deploy a blueprint and supply input parameters

# Target vRealize Automation System
$vrauri = 'https://&lt;vRealize Automation URL e.g. https://myvraserver.mydomain.local&gt;'
$user = '&lt;user name to authenticate to vRA&gt;'
$pass = (Get-Credential -UserName $user).GetNetworkCredential().password
$header = @{'Content-Type'='application/json'}

# Blueprint Deploy Parameters:
$environment        = 'Production'
$blueprintName      = '3-Tier App'
$deploymentName     = 'MyDeployment'
$description        = 'Deploying a vRA blueprint from code'
$appliancepassword  = 'MySecretPassword'
$department         = 'it'
$db_flavor          = 'medium'
$app_flavor         = 'small'
$web_flavor         = 'small'

try {   # Get an API refresh token
    $refres = Invoke-RestMethod -Method Post `
        -Uri "$vrauri/csp/gateway/am/api/login?access_token" `
        -Headers $header `
        -Body (@{'username'=$user;'password'=$pass} |
        ConvertTo-Json)
} catch { 
    Write-Host -ForegroundColor Red ("Error status: $_.Exception.Response.StatusCode.value__")
     Exit
}

try {   # Get an API access token
    $access_token = Invoke-RestMethod -Method Post `
        -Uri "$vrauri/iaas/api/login" `
        -Headers $header `
        -Body (@{'refreshToken'=$refres.refresh_token} | ConvertTo-Json)
} catch { 
    Write-Host -ForegroundColor Red ("Error status: $_.Exception.Response.StatusCode.value__")
    Exit 
}

# Get the projectid for deployment environment
$header = @{'Authorization'='Bearer ' + $access_token.token;'Content-Type'='*/*'}
$projects = (Invoke-RestMethod -Method Get `
    -Uri "$vrauri/project-service/api/projects" `
    -Headers $header).content
$projectid = ($projects | Where-Object { $_.name -eq $environment }).id
if (!$projectid) { 
    Write-Host -ForegroundColor Red "Environment $environment could not be located, exiting."
    Exit 
}

# Get the blueprintid for requested blueprint
$blueprints = (Invoke-RestMethod -Method Get `
    -Uri "$vrauri/blueprint/api/blueprints" `
    -Headers $header).content
$blueprintid = ($blueprints | Where-Object {$_.name -eq $blueprintName}).id
if (!$blueprintid) { 
    Write-Host -ForegroundColor Red "Blueprint $blueprintName could not be located, existing."
    Exit
}

# Create 'body' for API deployment request
$deploybody = @{
    'blueprintId'       =$blueprintid # found from API
    'deploymentName'    =$deploymentName
    'description'       =$description
    'projectId'         =$projectid # found from API
    'inputs'=@{
        'password'      =$appliancepassword
        'department'    =$department
        'db_flavor'     =$db_flavor
        'app_flavor'    =$app_flavor
        'web_flavor'    =$web_flavor
    }
}
$header = @{'Authorization'='Bearer ' + $access_token.token;'Content-Type'='application/json'}

try {   # Deploy the blueprint
    $res = Invoke-RestMethod -Method Post `
         -Uri "$vrauri/blueprint/api/blueprint-requests" `
         -Headers $header `
         -Body ($deploybody | ConvertTo-Json)
} catch {
    Write-Host -ForegroundColor Red ("Error status: $_.Exception.Response.StatusCode.value__")
    Exit 
}

if ($res.status -eq 'CREATED' -or $res.status -eq 'STARTED') {
    Write-Host ("Deployment request successfully submitted, deploymentId is $($res.deploymentId).")
} else {
    Write-Host -ForegroundColor Red ("Deployment request failed, returned data shown below.")
    $res
}
</pre>
</div>

 [1]: https://github.com/jakkulabs/PowervRA