---
title: Working with the vCloud API and PHP SDK – Part 1
author: Jon Waite
type: post
date: 2016-04-03T03:32:18+00:00
url: /2016/04/working-with-the-vcloud-api-and-php-sdk-part-1/
categories:
  - REST API
  - VMware
tags:
  - PHP
  - SDK
  - vCloud Director

---
# Introduction

While I love using the PowerCLI tools for manipulating vCloud Director, sometimes you need to perform actions that require hitting the API directly. Tools such as the <a href="https://addons.mozilla.org/en-US/firefox/addon/restclient/" target="_blank">RESTClient</a> plugin for Firefox, cURL and <a href="https://github.com/jkbrzt/httpie/" target="_blank">HTTPie</a> from the command line are good for interactive manipulation of the API, but what if you want to automate these API interactions?

Fortunately, rather than having to reinvent the wheel, VMware publish a variety of SDK&#8217;s (currently for PHP, Java and Microsoft .NET) which make this (relatively) straightforward, although sadly the documentation for these are lacking in basic configuration information which makes actually using them more problematic than it should be. They are still a better alternative than writing code directly against the HTTP API where you have to deal with decoding and re-encoding the XML objects used by the vCloud API itself directly.

In this series I&#8217;ll concentrate on the VMware PHP SDK for vCloud Director (PHP SDK). This first post will cover how to install and configure it. Once we have a working environment configured the following articles in this series will detail some (hopefully) useful scripts which use the PHP SDK to perform automation tasks against the vCloud API.

# Quick note on VMware PHP SDK versions

The link to download the PHP SDK for vCloud is on VMware&#8217;s site <a href="http://www.vmware.com/go/vcloudsdkforphp" target="_blank">here</a>, however if you work for a VMware Service Provider and have access to the vCloud Director for Service Providers code you&#8217;ll find a more recent version as follows:

Log in to your &#8216;<a href="https://my.vmware.com/" target="_blank">My VMware</a>&#8216; account and select the &#8216;View & Download Products&#8217; section. From the list select the &#8216;Download&#8217; link against the &#8216;vCloud Director for Service Providers&#8217; product.

Next select the &#8216;Drivers & Tools&#8217; tab and expand the &#8216;Automation Tools and SDK(s)&#8217; section, then select the Download link against the &#8216;VMware vCloud SDKs for Service Providers&#8217; item. This will take you to a page where you can download a later (v8.0.0 build 3010704 currently) version of the PHP SDK. (The direct link for this is <a href="https://my.vmware.com/group/vmware/details?downloadGroup=VCDSDK800&productId=532" target="_blank">here</a>, but I believe will only work if you have a valid Service Provider login for My VMware).

Download the .zip or .tar.gz version of the PHP SDK that suits your development environment, as far as I can tell the contents is identical between both versions so ease of unarchiving is the only difference. For the purposes of this series of posts it shouldn&#8217;t matter which version of the SDK (5.5 or 8.0) you use.

None of this makes sense to me &#8211; I know the vCloud Director product past version 5.5 is a service provider only offering, but since clients of vCloud Powered service providers are just as entitled to use the API as service providers then surely VMware should make the latest SDK available to everyone?

# Configuring your devlopment environment

There are 5 components involved in setting up a functioning development environment with the vCloud PHP SDK:

  * A working PHP installation (No web server required).
  * The PEAR modules &#8216;HTTP2_Request&#8217; and &#8216;Log&#8217;.
  * The PHP extensions &#8216;openssl&#8217; and &#8216;mbstring&#8217; added and enabled.
  * The VMware downloaded PHP SDK files.
  * A Configuration.ini file to control SDK logging.

Unfortunately only the first 2 of these are (partially) covered in the VMware documentation. The following sections detail how to configure a working development environment for all of these.

The documentation included on VMware&#8217;s site for the PHP SDK download mentions that the &#8216;Pear HTTP_Request2&#8217; package is required to use the PHP SDK, unfortunately it doesn&#8217;t mention that 2 additional extensions are also required (PEAR Log and the PHP mbstring). Without these additional packages you will either not be able to use the SDK at all, or receive strange error messages.

## PHP installation on Linux

If you are using a Linux platform with a package manager you will usually find a packaged PHP distribution available. On most CentOS systems this can be installed with:

`$ sudo yum install php`

On other Linux distributions the commands will vary so check the documentation for your particular environment, the <a href="http://php.net/manual/en/" target="_blank">PHP Documentation</a> has good installation instructions for a variety of platforms. You will also require Pear (PHP Extension and Application Repository framework) in order to be able to install the support packages required by the SDK. Again on CentOS this can be installed with:

`$ sudo yum install php-pear`

## Adding the required PHP extensions on Linux

Your Linux distribution should have an available package to install the mbstring (Multibyte Character support) extension for PHP, e.g. for CentOS Linux:

`$ sudo yum install php-mbstring`

To install the PEAR modules required for the vCloud PHP SDK:

`$sudo pear install HTTP_Request2 Log`

## PHP Installation on Windows

On Windows systems I would strongly advise using 64-bit Windows and the PHP version 7 releases from <a href="http://windows.php.net/download" target="_blank">http://windows.php.net/download</a> as this will support native PHP 64-bit integers on 64-bit platforms. This can be useful dealing with disk capacities (usually expressed in bytes) within the SDK which could overflow a 32-bit integer. (Note that even the 64-bit versions of PHP v5.x do not support 64-bit integers). For v7 on Windows you will also need to install the <a href="http://www.microsoft.com/en-us/download/details.aspx?id=48145" target="_blank">Visual C++ Redistributable for Visual Studio 2015</a> from Microsoft if you don&#8217;t already have it installed &#8211; make sure you install the version appropriate to your PHP environment (32-bit/64-bit).

Extract the downloaded PHP .zip file into a new folder (e.g. C:\PHP) and install the Visual C++ Redistributable appropriate to your version if needed.

Copy the &#8216;php.ini-development&#8217; file included in the download in this folder to &#8216;php.ini&#8217; &#8211; this will serve as the configuration point for your installation and will be used to enable extensions.

To install the PHP Pear extension on Windows, go to <a href="https://pear.php.net/manual/en/installation.getting.php" target="_blank">https://pear.php.net/manual/en/installation.getting.php</a> and download the &#8216;go-pear.phar&#8217; into the same folder you extracted PHP into. Install it from a command prompt opened into the same folder you extracted PHP into using:

`php go-pear.phar`

Note: if you are not using an &#8216;administrator&#8217; command prompt you will need to change the default path for &#8216;pear.ini&#8217; to something other than C:\Windows &#8211; I just placed it in the PHP directory (C:\PHP\pear.ini).

You will need to double-click the &#8216;PEAR_ENV.reg&#8217; file to add appropriate environment variables for PEAR to your user account to the Windows registry. You should also add the directory you extracted PHP into to your Windows PATH &#8211; this is under Windows &#8216;Advanced System Settings&#8217;. Note that you will need to re-open a command prompt for PATH changes to take effect.

## Adding required PHP extensions on Windows

To install the &#8216;HTTP_Request2&#8217; and &#8216;Log&#8217; modules from a Windows Command prompt type:

`pear install HTTP_Request2 Log`

The mbstring (multibyte) extension is usually present (in the &#8216;ext&#8217; subdirectory of your PHP installation) as &#8216;php_mbstring.dll&#8217; on Windows, but not enabled by default. The same is true for the openssl extension. To enable these extensions edit your php.ini file and find the lines:

`;extension=php_mbstring.dll<br />
:::<br />
;extension=php_openssl.dll`

And remove the leading semi-colons (;) then save the file. You can test whether the modules are loaded correctly by typing &#8216;php -m&#8217; and checking that &#8216;mbstring&#8217; and &#8216;openssl&#8217; are listed in the output in the [PHP Modules] section.

# VMware PHP SDK Files

The PHP SDK itself can be extracted to any convenient folder &#8211; on Linux I&#8217;d suggest under your home directory and on Windows pick somewhere easy to locate (e.g. C:\PHPSDK). Ensure that the folder structure remains intact &#8211; you should have 3 subdirectories in the extracted SDK named &#8216;docs&#8217;, &#8216;library&#8217; and &#8216;samples&#8217;. The essential one (and only one required to actually use the SDK) is the &#8216;library&#8217; folder.

# Configuration.ini (pear/Log configuration file)

The Vmware documentation makes no mention of it, but the &#8216;ServiceAbstract.php&#8217; file included in the library file (/library/VMware/VCloud/ServiceAbstract.php) relies on both the Pear Log module (which we installed) and a text file &#8216;Configuration.ini&#8217; which is read at various points in the file to determine logging options for API interactions.

This is very useful (once you know about it), but the file itself is not supplied with the PHP SDK download and must be manually created. To do this, create a new text file in the folder where you will be working with the SDK (e.g. C:\PHPSDK) named &#8216;Configuration.ini&#8217; and specify the contents as:

`[log_section]<br />
` `log_handler_name=file<br />
` `log_file_location=phpsdk.log<br />
` `log_level=PEAR_LOG_DEBUG`

This will log all API interactions to a text file (phpsdk.log) in the current directory and is incredibly useful for troubleshooting &#8211; you can control the verbosity of the logging by changing the log_level parameter (as well as the log filename and type using the other options). For full documentation on available options and values see the <a href="http://pear.github.io/Log/" target="_blank">Pear Log documentation</a>.

# config.ini (VMware PHP SDK configuration file)

The last piece of configuration required is to copy the file &#8216;config.ini&#8217; from the PHP SDK &#8216;samples&#8217; folder into the location where you will be developing your PHP code. You should edit the copied file and ensure that the section that reads:

`set_include_path(implode(PATH_SEPARATOR, array('.','../library',get_include_path(),)));`

Is updated to refer to the location where your PHP SDK &#8216;library&#8217; folder exists. For example, on Windows if the PHP SDK is extracted to C:\PHPSDK then this line should be updated to read:

`set_include_path(implode(PATH_SEPARATOR, array('.','C:\PHPSDK\library',get_include_path(),)));`

To check whether everything is working, try creating a file &#8216;test.php&#8217; in the root of your working directory (where you&#8217;ve edited config.php and saved Configuration.ini)

`<?php<br />
` `  include './config.php';<br />
` `?>`

If you&#8217;ve configured everything correctly then running this from a command prompt `php test.php` should return with no output or errors.

**Note:** If you are using PHP 7 on Windows you may get a warning from the Log.php file included by the SDK which looks similar to:

`PHP Deprecated: Methods with the same name as their class will not be constructors in a future version of PHP; Log has a deprecated constructor in C:\PHP\pear\Log.php on line 38`

This is harmless and safe to ignore, but if it annoys you and you want to prevent it being displayed you can add a new line to the Log.php file specified in the warning message as below just prior to the &#8216;public static function factory($handler, $name = &#8221;, $ident = &#8221;,&#8230;) line:

`public function __construct(){}`

You will proabbly also need to add this line after the &#8216;class&#8217; definition in the pear/Log/file.php file.

In the next parts of this series I&#8217;ll be using this environment to show you how to achieve some useful interactions with a vCloud platform using the Perl SDK, I&#8217;ll update this post with links once these are posted. As always, please leave any feedback in the comments, I try to answer as much as I can.

Jon.