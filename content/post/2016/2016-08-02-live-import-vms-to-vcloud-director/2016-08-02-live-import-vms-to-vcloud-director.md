---
title: Live import VMs to vCloud Director
author: Jon Waite
type: post
date: 2016-08-01T23:51:37+00:00
url: /2016/08/live-import-vms-to-vcloud-director/
categories:
  - PowerShell
  - REST API
  - VMware
tags:
  - PowerCLI
  - PowerShell
  - REST API
  - vCloud Director
  - vCloud Director 8
  - VMware

---
Tom Fojta wrote a [great blog post][1] about the new capability in vCloud Director 8.10 to import running VMs into vCloud Director. This is a huge asset in migration scenarios where customers can't afford outages when being migrated into the vCD environment. Unfortunately the API syntax to actually initiate the import is a little convoluted and not the easiest process to manage.

I set about writing a PowerShell script to significantly simplify the process of initiating a live-import operation. The script itself is available from github at the following link: <https://github.com/jondwaite/vcdliveimport>.

The liveimport.ps1 script contained in this repository does the following:

  * Prompts for a credential to be used to connect to both vCloud Director (System context) and vCenter - if you have different usernames/passwords for each you'll need to adjust this.
  * Enumerates the available vCenter instances registered as Provider Virtual Datacenters (PVDCs) in vCloud Director and allows one to be selected as the source vCenter for the migration.
  * Lists the available VMs in the selected vCenter instance, filters this list based on selectable criteria (e.g. don't offer to import 'Guest Introspection' VMs) and allows the source VM to be selected.
  * Lists available destination Virtual Datacenters (VDCs) in the vCloud Director environment and allows the destination VDC to be selected.
  * Displays the appropriate POST request information to be submitted to vCloud Director to initiate the live-import of this VM.
  * Optionally - Submits the REST API request directly to the vCloud Director environment to actually initiate the import process.

An example transcript of this process is show below. Hopefully this helps someone else out and helps to make it easier for you to live-import running VMs into vCloud Director.

Jon.

Example Session Transcript:

<pre class="font-size:12 plain:true lang:ps decode:true ">PowerCLI C:\> liveimport.ps1
Connected to vCloud Director OK
vCenter(s) Found:
-----------------
vc01 (https://vcd01.dev.local/api/admin/extension/vimServer/158f73ec-a999-4332-8250-f4dd5e6c4971)

Selecting vc01 as only pVDC vCenter found.
vCenter vc01 selected.
Connected to vCenter Server vc01 OK

Evaluating vCenter VMs as migration candidates.............
Available candidate VMs:
------------------------
testvm01

Enter VM name to live-migrate to vCloud (or 'quit' to exit): testvm01
Selected VM testvm01 for live import to vCloud.

Available VDCs:
---------------
Lab VDC
Enter Destination VDC Name (or quit to exit): Lab VDC
VDC 'Lab VDC' selected.

URI for POST operation:
https://vcd01.dev.local/api/admin/extension/vimServer/158f73ec-a999-4332-8250-f4dd5e6c4971/importVmAsVApp

XML Document Body:
<?xml version="1.0" encoding="UTF-8"?>
<ImportVmAsVAppParams xmlns="http://www.vmware.com/vcloud/extension/v1.5" name="testvm01" sourceMove="true">
 <VmMoRef>vm-111</VmMoRef>
 <Vdc href="https://vcd01.dev.local/api/admin/vdc/b21c92fa-1a6f-42a9-8c1e-ef0947c7be76" />
</ImportVmAsVAppParams>

Content-Type: application/vnd.vmware.admin.importVmAsVAppParams+xml

Would you like to submit this API request to live import this VM? (y or n) (or quit to exit): y
Making request to live import VM testvm01...
Response was:
<?xml version="1.0" encoding="UTF-8"?>
<VApp xmlns="http://www.vmware.com/vcloud/v1.5" ovfDescriptorUploaded="true" deployed="false" status="0" name="testvm01" id="urn:vcloud:vapp:ff1a9021-2c21-4378-99c7-b77bdaf3e9e6" href="https://vcd01.ndev.local/api/vApp/vapp-ff1a9021-2c21-4378-99c7-b77bdaf3e9e6" type="application/vnd.vmware.vcloud.vApp+xml" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.vmware.com/vcloud/v1.5 http://vcd01.dev.local/api/v1.5/schema/master.xsd">
 <Link rel="down" href="https://vcd01.dev.local/api/network/251f72a8-19b5-4e7d-94ba-0523ee919cd9" name="VM Network" type="application/vnd.vmware.vcloud.vAppNetwork+xml" />
 <Link rel="down" href="https://vcd01.dev.local/api/vApp/vapp-ff1a9021-2c21-4378-99c7-b77bdaf3e9e6/controlAccess/" type="application/vnd.vmware.vcloud.controlAccess+xml" />
 <Link rel="up" href="https://vcd01.dev.local/api/vdc/b21c92fa-1a6f-42a9-8c1e-ef0947c7be76" type="application/vnd.vmware.vcloud.vdc+xml" />
 <Link rel="down" href="https://vcd01.dev.local/api/vApp/vapp-ff1a9021-2c21-4378-99c7-b77bdaf3e9e6/owner" type="application/vnd.vmware.vcloud.owner+xml" />
 <Link rel="down" href="https://vcd01.dev.local/api/vApp/vapp-ff1a9021-2c21-4378-99c7-b77bdaf3e9e6/metadata" type="application/vnd.vmware.vcloud.metadata+xml" />
 <Link rel="ovf" href="https://vcd01.dev.local/api/vApp/vapp-ff1a9021-2c21-4378-99c7-b77bdaf3e9e6/ovf" type="text/xml" />
 <Link rel="down" href="https://vcd01.dev.local/api/vApp/vapp-ff1a9021-2c21-4378-99c7-b77bdaf3e9e6/productSections/" type="application/vnd.vmware.vcloud.productSections+xml" />
 <Tasks>
 <Task cancelRequested="false" expiryTime="2016-10-31T11:27:25.312+13:00" operation="Importing Virtual Application testvm01(ff1a9021-2c21-4378-99c7-b77bdaf3e9e6)" 
operationName="importSingletonVapp" serviceNamespace="com.vmware.vcloud" startTime="2016-08-02T11:27:25.312+12:00" status="queued" name="task" id="urn:vcloud:task:37576f8c-0d39-444c-8842-81436ddb421e" href="https://vcd01.dev.local/api/task/37576f8c-0d39-444c-8842-81436ddb421e" type="application/vnd.vmware.vcloud.task+xml">
 <Owner href="https://vcd01.dev.local/api/vApp/vapp-ff1a9021-2c21-4378-99c7-b77bdaf3e9e6" name="testvm01" type="application/vnd.vmware.vcloud.vApp+xml" />
 <User href="https://vcd01.dev.local/api/admin/user/fa7fb40e-648c-4787-ad0f-711dcc315261" name="system" type="application/vnd.vmware.admin.user+xml" />
<Organization href="https://vcd01.dev.local/api/org/3f46b162-f794-4d9e-8fa1-6ca1ee7b6377" name="Lab" type="application/vnd.vmware.vcloud.org+xml" />
 <Progress>1</Progress>
 <Details />
 </Task>
 </Tasks>
 <DateCreated>2016-08-02T11:27:25.079+12:00</DateCreated>
 <Owner type="application/vnd.vmware.vcloud.owner+xml">
 <User href="https://vcd01.dev.local/api/admin/user/1545cb33-9151-43e2-a156-9d20e3b966c0" name="system" type="application/vnd.vmware.admin.user+xml" />
 </Owner>
 <InMaintenanceMode>false</InMaintenanceMode>
</VApp></pre>

&nbsp;

 [1]: https://fojta.wordpress.com/2016/05/27/import-running-vm-to-vcloud-director/