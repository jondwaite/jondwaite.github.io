---
title: Invoke-vCloud PowerShell Module
author: Jon Waite
type: post
date: 2017-09-08T22:42:54+00:00
url: /2017/09/invoke-vcloud-powershell-module/
categories:
  - PowerShell
  - REST API
  - VMware
tags:
  - Module
  - PowerCLI
  - PowerShell
  - REST API
  - vCloud Director 8

---
Several of the posts on here and many of the internal PowerShell projects I use at work require direct interaction with the vCloud Director REST API. Usually this is because features exposed in the API aren't yet directly implemented as PowerCLI cmdlets. A good example would be the modules I wrote to allow manipulation of independent disks with vCD VMs here: [Independent Disks in vCloud via PowerCLI][1].

As I wrote more and more scripts that require interaction with the vCD API I started to develop a generic function to allow easier interaction with it. I'm intending in future to use this as the basis for future scripts requiring this functionality which should be useful in keeping the script size smaller and provide an easier method of interacting with the API.

PowerShell has a built-in 'Invoke-RestMethod' call which submits a REST request and gathers any responses, but there are a variety of additional parameters required in a typical vCD API call which this method doesn't know about and which need to be supplied with each call. Wrapping Invoke-RestMethod in a function allows us to much more easily consume the API and makes handling calls which return a long-running task much easier to handle by giving the option to return immediately or to wait for the complete interaction to complete before returning.

Typically to call the API we need to provide the following:

|Item|Description|
|---|---|
|URI|The URI (Uniform Resource Identifier) for the API call (typically of the form https://my.cloud.com/api/requestpath).|
|Method|This is simply which HTML method we are invoking from the common HTML verbs ('GET','PUT','POST','DELETE') - the fifth verb ('PATCH') doesn't appear to be used much (if at all) in the vCloud API, but could be specified if required. We can also assume a sensible/safe default value of 'GET' since that will only read from the API and not change anything.|
|Authorization|Provided as an HTML Header using the x-vcloud-authorization tag and a previously existing session ID. We could supply the session ID as part of the call, but it's usually much more convenient to attempt to match the request URI against PowerShell's existing global view of connected vCD API endpoints and use the existing session ID available from here.|
|Accept|Provided as part of the HTML header, this tag specifies the type of information we are prepared to accept back from the API, this will usually be set to 'application/*+xml' but we also need to specify which version of the API we wish to use - in vCloud Director 8.20 this is version 27.0 so the complete Accept token will be 'application/\*+xml;version=27.0'. We can provide a sensible default value for this in our module so that if it is ommitted we still get a sensible version specification.|
|ContentType|When we are sending data to the API with a PUT or POST request, we need to specify the type of document we are sending using the ContentType header, for GET requests this isn't required.|
|Body|When sending data to the API, this will be the (usually XML formatted) document body.|
|Timeout|Some API operations can take a while to complete, specifying a timeout value allows our script to carry on or raise an error if an API call is taking an excessively long time.|

In addition to these, I've also included a flag 'WaitforTask' which can either be true or false. If set to 'true' and if the result of the API call is a 'Task' object reference then the script will monitor the task and wait until it has completed (or timeout exceeded) and then return the task status to our script. This can be useful for operations that can take considerable time when it doesn't make sense for your script to continue until the previous action has been completed - e.g. cloning a new vApp and then modifying it's network settings.

I was keen to explore using [Microsoft's PowerShell Gallery][2] as a mechanism to distribute modules, this should make it easier to be able to use these modules and functions wherever needed by simply importing the module as required. The process to make a module available in the gallery is reasonably straightforward and documented on the site, but basically once you've signed up and got an API key you simply call Publish-Module and provide the path to your module code and manifest including your API key as an identifier.

So I'm pleased to announce that the initial release version of my [Invoke-vCloud][3] module is now available from the PowerShell Gallery and can be included in any scripts using:

```
Install-Module -Name Invoke-vCloud -Scope CurrentUser
```

It can also be downloaded/inspected using:

```
Save-Module -Name Invoke-vCloud -Path <path>
```

Of course if running an administrative PowerShell instance it can also be installed for all users on the system using:

```
Install-Module -Name Invoke-vCloud
```

Once installed, it can be used in your scripts by specifying the required URI parameter and any optional specifications:

```
PS C:\> Invoke-vCloud -URI https://my.cloud.com/api/org

xml OrgList
--- -------
version="1.0" encoding="UTF-8" OrgList
```

You will either need to have an existing PowerCLI vCloud session (Connect-CIServer), or have an x-vcloud-authorization token for a valid/authenticated session which you can pass to Invoke-vCloud using the -vCloudToken parameter.

I've included parameter descriptions in the module to assist which can be viewed using the Get-Help cmdlet:

```
PS C:\> Get-Help Invoke-vCloud -detailed

NAME
    Invoke-vCloud

SYNOPSIS
    Provides a wrapper for Invoke-RestMethod for vCloud Director API calls


SYNTAX
    Invoke-vCloud [-URI]  [[-ContentType] ] [[-Method] ] [[-ApiVersion] ] [[-Body] ] [[-Timeout] ] [[-WaitForTask] ] [[-vCloudToken] ] []


DESCRIPTION
    Invoke-vCloud provides an easy to use interface to the vCloud Director
    API and provides sensible defaults for many parameters. It wraps the native
    PowerShell Invoke-RestMethod cmdlet.


PARAMETERS
    -URI 
        A mandatory parameter which provides the API URI to be accessed.

    -ContentType 
        An optional parameter which provides the ContentType of any submitted data.

    -Method 
        An optional parameter to specify the HTML verb to be used (GET, PUT, POST or
        DELETE). Defaults to 'GET' if not specified.

    -ApiVersion 
        An optional parameter to specify the API version to be used for the call.
        Defaults to '27.0' if not specified which is the API version for vCloud
        Director v8.20.

    -Body 
        An optional parameter which specifies XML body to be submitted to the API
        (usually for a PUT or POST action).

    -Timeout 
        An optional parameter which specifies the time (in seconds) to wait for an API
        call to complete. Defaults to 40 seconds if not specified.

    -WaitForTask 
        If the API call we submit results in a Task object indicating an asynchronous
        vCloud task, should we wait for this to complete before returning? Defaults to
        $false.

    -vCloudToken 
        An alternative method of passing a session token to Invoke-vCloud if there is
        no current PowerCLI session established to a vCloud endpoint. The session must
        have already been established and be still valid (not timed-out). The value
        supplied is copied to the 'x-vcloud-authorization' header value in API calls.
```


I have some new scripts in development against some of the new features in the recently announced [vCloud Director v9][4] which I'm intending to use this module for and will post as soon as v9 is released.

As always I welcome any comments and suggestions for improving this module.

Jon.

 [1]: /2017/02/independent-disks-in-vcloud-via-powercli/
 [2]: https://www.powershellgallery.com/
 [3]: https://www.powershellgallery.com/packages/Invoke-vCloud/1.0.0
 [4]: https://blogs.vmware.com/vcloud/2017/08/vmware-announces-new-vcloud-director-9-0.html