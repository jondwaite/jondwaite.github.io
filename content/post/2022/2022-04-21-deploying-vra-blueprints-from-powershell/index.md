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
The [PowervRA][1] module is a great set of PowerShell cmdlets for managing a VMware vRealize Automation (vRA) environment, but I wanted to learn a bit more about the REST API available from vRA and also had a requirement - to script a deployment from an existing vRA blueprint ('Cloud Template') which had no PowervRA cmdlet. So I set about writing my own script in PowerShell to do this and have documented this and the steps required in this post.

The steps in this process are:

- Gather the parameters and inputs for the deployment
- Authenticate to the vRA API and obtain a refresh token
- Use the obtained refresh token to obtain an access token
- Find the Project Id and Blueprint Id that we want to deploy from the API
- Request the deployment and check the request is submitted successfully

I also wanted to provide some basic error checking so that if the specified vRA Project or Blueprint could not be found the script provided meaningful feedback.

### Gather the parameters and inputs for the deployment

In the first code block, I simply configure variables for the environment - including prompting for the password which will be used to authenticate to vRA:

```
# Target vRealize Automation System
$vrauri = 'https://<vRealize Automation URL e.g. https://myvraserver.mydomain.local>'
$user = '<user name to authenticate to vRA>'
$pass = (Get-Credential -UserName $user).GetNetworkCredential().password
$header = @{'Content-Type'='application/json'}
```

The next block defines the parameters for the blueprint that will be deployed, including which vRA Project will be the deployment target, which blueprint will be used and the input parameters required for deploying this instance. The blueprint I'm using is a 3-tier application that I've been working on to provide a more realistic demonstration of vRA capabilities. Obviously these parameters will vary depending on the environment and template being deployed

```
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
```
### Authenticate to the vRA API and obtain a refresh token

Next we need to POST the authentication data to the `/csp/gateway/am/api/login?access_token` URL to obtain a refresh token:

```
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
```

I've included some basic error handling here so that if an error occurs (e.g. an incorrect user name or password is provided) this is caught and the script gracefully exits. ConvertTo-Json is used to ensure our parameters are correctly provided to the API.

### Use the obtained refresh token to obtain an access token

Once we have our refresh token, we can call the `/iaas/api/login` URL to obtain the bearer token which will be used to authenticate all our subsequent API requests.

```
try {   # Get an API access token
    $access_token = Invoke-RestMethod -Method Post `
        -Uri "$vrauri/iaas/api/login" `
        -Headers $header `
        -Body (@{'refreshToken'=$refres.refresh_token} | ConvertTo-Json)
} catch { 
    Write-Host -ForegroundColor Red ("Error status: $_.Exception.Response.StatusCode.value__")
    Exit 
}
```

### Find the Project Id and Blueprint Id that we want to deploy from the API

If everything has gone ok up to this point, we now have a token (in `$access_token.token`) which can be used to authenticate our next requests. In order to be able to request deployment of our blueprint we need to provide both the Project Id and Blueprint Id to the API. As I didn't want to have to keep looking up and coding these Id's into scripts I've used 2 calls to search and match these by name (`$environment` is the 'friendly' Project name and `$blueprintName` is the 'friendly' blueprint name in this example). Since both API calls use the same headers, we can set up a single `$header` variable and use it for both calls containing our authorization token.

```
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
```

### Request the deployment and check the request is submitted successfully

Now we have all of the information required, we can build a JSON object containing all our inputs to the deployment process and then make an API POST request to /blueprint/api/blueprint-requests to initiate the actual deployment.

```
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
```

### Summary

Hopefully this post is detailed enough that it will be easy to adjust for your particular blueprints and requirements if needed. I've included the entire script in one block below for easier copying/pasting if you want to try this in your own environment.

As always, comments, feedback and suggestions for improvement are welcome.

Jon

```
# Complete code listing - vRA automation to deploy a blueprint and supply input parameters

# Target vRealize Automation System
$vrauri = 'https://<vRealize Automation URL e.g. https://myvraserver.mydomain.local>'
$user = '<user name to authenticate to vRA>'
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
```

 [1]: https://github.com/jakkulabs/PowervRA