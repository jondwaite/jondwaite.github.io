---
title: Using vCloud Director PowerCLI and vcd-cli with Federated User Accounts
author: Jon Waite
type: post
date: 2018-03-10T10:30:24+00:00
url: /2018/03/using-vcloud-director-powercli-and-vcd-cli-with-federated-user-accounts/
categories:
  - PowerShell
  - vCloud Director
tags:
  - PowerCLI
  - PowerShell
  - Python
  - vcd-cli
  - vCloud Director

---
One of the issues that vCloud Director user can run into is user authentication when using the PowerCLI and <a href="https://github.com/vmware/vcd-cli" target="_blank" rel="noopener">vcd-cli</a> tools to manage their cloud deployments. For ‘Local’ user accounts defined in the vCloud Director portal this isn’t an issue as username/password are stored in the vCD database and can be directly authenticated. However, many customers want to federate their vCloud users with an external directory service (often Microsoft AD FS or other similar service). Typically this is done so that security groups in the external directory can be used to control access levels, and so that additional authentication mechanisms like 2-Factor Authentication (2FA) can be applied to accounts.

If you attempt to use CLI tools like vcd-cli or PowerCLI to authenticate with a federated user account you will get a ‘Login Failed’ or ‘Unauthorized’ failure and won’t be able to connect to the service.

Fortunately, both vcd-cli and PowerCLI allow you to use an existing browser vCloud session ID to connect to the vCD API. To use this you connect to your vCloud portal in a web browser and then then use your browser’s tools to find the session ID for your connection. Once you have the session ID you can create a PowerCLI or vcd-cli session using that token.

It can sometimes be easier to use a browser plugin or extension to help find the session ID, ones which show session cookies and/or HTTP headers work best, but even without these it is possible.

In Google Chrome for example, use <ctrl + shift + I> (or Menu / More Tools / Developer Tools) to open the developer interface. Next click on the ‘Network’ heading at the top of the developer panel and refresh the vCloud Director portal. Scroll down to one of the ‘amfsecure’ document lines and select the ‘Headers’ tab, you should see a panel similar to this:

[<img loading="lazy" decoding="async" class="aligncenter" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/03/image_thumb.png" alt="image" width="560" height="952" border="0" />][1]

You can simply copy the value from the highlighted entry (87489f6a17044d66bc36704ce5c4e45c in this example) and use that to establish a vcd-cli or PowerCLI session:

For vcd-cli:

`vcd login <cloud endpoint> <org name> <user name> –d <session ID string>`

e.g.

`vcd login mycloudprovider.com myorg joebloggs –d 87489f6a17044d66bc36704ce5c4e45c`

For PowerCLI:

`Connect-CIServer –Server <cloud endpoint> –SessionID <session ID string>`

e.g.

`Connect-CIServer –Server mycloudprovider.com –SessionID 87489f6a17044d66bc36704ce5c4e45c`

You will then be connected as the same user from your browser session and able to run all the PowerCLI or vcd-cli commands with that user account.

## An easier way?

Rather than digging around for HTTP headers and cookies in a browser, vcd-cli has a built-in module which is meant to retrieve the sessionID from a browser session automatically and use this to authenticate, the syntax is:

`vcd login session list chrome`

Which should return the session ID from an instance of Chrome, but in my initial testing this was not returning any output at all.

Reading through the vcd-cli sources it appears that this option relies on a Python extension ‘browsercookie’ which can be installed using `pip install --user browsercookie`. Browsercookie has a dependency on the ‘pycrypto’ module which must also be installed. However, even with both pycrypto and browsercookie installed I couldn’t get this option to work.

I did manage to get this working by installing the `browser_cookie3` module from [https://pypi.python.org/pypi/browser-cookie3/0.6.0][2] by using `pip install --user browser-cookie3` and then making the following changes in the vcd-cli\login.py file:

Line 24: Change:

`from vcd_cli import browsercookie` to: `import browser_cookie3`

On both lines 126 and 148: Change:

`cookies = browsercookie.chrome()` to: `cookies = browser_cookie3.chrome()`

Once these changes are complete the ‘vcd login session list chrome’ command can be used to obtain the current session ID from Chrome automatically:

[<img loading="lazy" decoding="async" class="aligncenter" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/03/image_thumb-1.png" alt="image" width="478" height="77" border="0" />][3]

And this can be used directly to login automatically once a Chrome session exists using the `--use-browser-session` switch.

Also note that you can obtain the session ID like this from vcd-cli and use it to authenticate a PowerCLI session with no issues at all.

Jon.

 [1]: https://kiwicloud.ninja/wp-content/uploads/2018/03/image.png
 [2]: https://pypi.python.org/pypi/browser-cookie3/0.6.0 "https://pypi.python.org/pypi/browser-cookie3/0.6.0"
 [3]: https://kiwicloud.ninja/wp-content/uploads/2018/03/image-1.png