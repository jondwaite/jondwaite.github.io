---
title: Using VMware Container Service Extension (CSE)
author: Jon Waite
type: post
date: 2018-02-24T22:26:21+00:00
url: /2018/02/using-vmware-container-service-extension-cse/
categories:
  - Containers
  - vCloud Director
  - VMware
tags:
  - Containerisation
  - CSE
  - Docker
  - Kubernetes
  - Python
  - vcd-cli
  - vCloud Director

---
[Yesterday][1] I wrote showing the currently available container hosting options from VMware. As we’ve recently deployed one of these options – CSE in our environment I thought it would be useful to show a sample workflow on how the service functions and how customers can use this to deploy and manage both CSE clusters, and also micro-service applications onto those clusters.

There are a few requirements on the tenant side which must be completed prior to any of this working:

  * An Organizational Administrator login to the vCloud platform where CSE is deployed.
  * Access to a virtual datacenter (VDC) with sufficient CPU, Memory and Storage resources for the cluster to be deployed into.
  * An Org VDC network which can be used by the cluster and has sufficient free IP addresses in a Static Pool to allocate to the cluster nodes (clusters take 1 IP address for the ‘master’ node and an additional address for each ‘worker’ node deployed).
  * A client prepared with Python v3 installed and the vcd-cli and container-service-extension packages installed on it.
  * The {$HOMEDIR}\.vcd-cli\profiles.yaml file edited to add the CSE extension to vcd-cli.
  * The kubectl utility installed to administer the Kubernetes cluster once deployed and working. kubectl can be obtained most easily from <a href="https://kubernetes.io/docs/tasks/tools/install-kubectl/" target="_blank" rel="noopener">here</a>.

Detailed instructions for the client setup can be found in the CSE documentation at [https://vmware.github.io/container-service-extension/#tenant-installation][2]. Note that on a Windows platform the .vcd-cli folder and profiles.yaml file will not be automatically created, but you can do this manually by

<pre>mkdir %HOMEPATH%\.vcd-cli</pre>

from a DOS prompt and then using vcd-cli to log in and out of your cloud provider. This will cause profiles.yaml to be generated in the .vcd-cli folder. The profiles.yaml file can then be edited in your favourite text editor to add the required CSE extension lines.

### Deploying a Cluster with CSE

When deploying a cluster, you will need to know the storage profile and network names which the cluster will use, the easiest way of obtaining these is either from the vCloud portal, or using the vcd vdc info command when logged in to your environment:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-20.png" alt="image" width="547" height="536" border="0" />][3]

If you have multiple VDCs available to you, the ‘vcd vdc use <VDC Name>’ command to set which one to work with.

In this example we will be using the highlighted entries (the ‘Tyrell-Servers’ network and the ‘CHC Performance’ storage profile).

To retrieve a list of available cluster deployment templates that the Service Provider has made available to us we can use the vcd cse template list command:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image4_thumb-1.png" alt="image" width="722" height="119" border="0" />][4]

In this example only the Photon OS template is available and is also the default template. CSE actually comes with 2 profiles (Photon OS v2 and Ubuntu Linux 16-04, but I’ve only installed the Photon OS v2 template in my lab environment). The default template will be used if you do not specify the ‘&#8211;template’ switch when creating a cluster.

The cluster create command takes a number of parameters which are documented in the CSE page:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image12_thumb-1.png" alt="image" width="850" height="328" border="0" />][5]

Be careful with the memory specification is it is in MB and not GB.

I chose to generate a public/private key to access the cluster nodes without needing a password, but this is optional. If you want to use key authentication you will need to generate a key pair and specify the public key filename in the cluster creation command using the &#8211;ssh-key switch.

To deploy a cluster with 3 worker nodes into our VDC where each node has 4GB of RAM and 2 CPUs using my public key and the network and storage profile identified above:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image1_thumb.png" alt="image" width="1282" height="121" border="0" />][6]

The deployment process will take several minutes to complete as the cluster VMs are deployed and started.

In to the vCloud Director portal, we can see the new vApp that has been deployed with our master and worker nodes inside it, we can also see that all 4 VMs are connected to the network we specified:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image6_thumb.png" alt="image" width="1028" height="575" border="0" />][7]

To see the details of the nodes deployed we can use ‘vcd cse node list <cluster name>’:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image11_thumb.png" alt="image" width="629" height="123" border="0" />][8]

To manage the cluster with kubectl, we need a configuration file for Kubernetes containing our authentication certificates. kubectl by default looks for a file named ‘config’ in a folder called ‘.kube’ under the current user’s home directory. The config file itself can be downloaded using CSE. To create the folder and write the config file:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image15_thumb.png" alt="image" width="493" height="61" border="0" />][9]

If you have multiple deployed clusters you can create separate config files for each one (with different file names) and use the &#8211;kubeconfig= switch to kubectl to select which one to use.

To test kubectl we can ask for a list of all containers (‘pods’ in Kubernetes) from the cluster, the ‘&#8211;all-namespaces’ switch shows system pods as well as any user created pods (which we don’t have yet). This must be run from a machine that has network connectivity with the deployed nodes (the ‘Tyrell-Servers’ network in this example):

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image20_thumb.png" alt="image" width="689" height="253" border="0" />][10]

&nbsp;

### Cluster Scaling

#### Adding Nodes to Clusters

If we need to add worker nodes to a cluster this is accomplished with the ‘vcd cse node create’ command. For example, we can add a 4th worker node to our ‘myCluster’ cluster as follows:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-33.png" alt="image" width="1204" height="76" border="0" />][11]

The node list now shows our cluster with 4 worker nodes including our new one:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-22.png" alt="image" width="630" height="140" border="0" />][12]

#### Removing Nodes from Clusters

To remove a cluster member is just as easy using the ‘vcd cse node delete’ command:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-34.png" alt="image" width="1112" height="111" border="0" />][13]

You will be prompted to confirm the node deletion, and if you have deployed container applications you should ensure that the node is properly drained and/or replica sets and deployments configured correctly so that the node deletion will not impact your applications.

&nbsp;

### Cluster Host Affinity

One item that CSE does not deal with yet is creating vCloud Anti-Affinity rules to ensure that your worker nodes are spread across different physical hosts. This means that with appropriately configured applications a host failure will not impact on the availability of your deployed services. It is reasonably straightforward to add anti-affinity rules in the vCloud portal though.

Our test cluster is back to 3 nodes following the deletion example:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-24.png" alt="image" width="629" height="132" border="0" />][14]

In the vCloud portal we can go to ‘Administration’ and select our virtual datacenter in the left pane, we will then see an ‘Affinity Rules’ tab:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-35.png" alt="image" width="1028" height="504" border="0" />][15]

Clicking the ‘+’ icon under Anti-Affinity Rules allows us to create a new rule to keep our worker nodes on separate hosts:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-26.png" alt="image" width="646" height="594" border="0" />][16]

Provided the VDC has sufficient backing physical hosts, the screen will update to show the new rule and that it has successfully been applied and separated the worker nodes to different hosts:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-36.png" alt="image" width="1030" height="226" border="0" />][17]

Of course if the host running the master node experiences a failure then this will be unavailable until the VMware platform restarts the VM on another host.

&nbsp;

### Application Deployment using kubectl

Of course now that our cluster is up and running, it would be nice to actually deploy a workload to it. The ‘sock shop’ example mentioned in the CSE documentation is a good example application to try as it consists of several pods running in a separate namespace.

First we use kubectl to create the namespace:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-28.png" alt="image" width="389" height="43" border="0" />][18]

Now we can deploy the application into our name space from the microservices-demo project on github. You can read more about the sock-shop demo app at [https://github.com/microservices-demo/microservices-demo][19].

`C:\Users\jon>kubectl apply -n sock-shop -f "<a href="https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"">https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"</a><br />
deployment "carts-db" created<br />
service "carts-db" created<br />
deployment "carts" created<br />
service "carts" created<br />
deployment "catalogue-db" created<br />
service "catalogue-db" created<br />
deployment "catalogue" created<br />
service "catalogue" created<br />
deployment "front-end" created<br />
service "front-end" created<br />
deployment "orders-db" created<br />
service "orders-db" created<br />
deployment "orders" created<br />
service "orders" created<br />
deployment "payment" created<br />
service "payment" created<br />
deployment "queue-master" created<br />
service "queue-master" created<br />
deployment "rabbitmq" created<br />
service "rabbitmq" created<br />
deployment "shipping" created<br />
service "shipping" created<br />
deployment "user-db" created<br />
service "user-db" created<br />
deployment "user" created<br />
service "user" created`

We can see deployment status by getting the pod status in our namespace:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-29.png" alt="image" width="623" height="251" border="0" />][20]

After a short while all the pods should have been created and show a status of ‘Running’:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-30.png" alt="image" width="541" height="253" border="0" />][21]

The ‘sock-shop’ demo creates a service which listens on port 30001 on all nodes (including the master node) for http traffic, so we can get our master node IP address from ‘vcd cse node list myCluster’ and open this page in a browser:

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-31.png" alt="image" width="633" height="124" border="0" />][22]

And here’s our deployed application running!

[<img loading="lazy" decoding="async" style="display: inline; background-image: none;" title="image" src="https://kiwicloud.ninja/wp-content/uploads/2018/02/image_thumb-32.png" alt="image" width="1028" height="725" border="0" />][23]

### Summary / Further Reading

Of course there’s much more that can be done with Docker and Kubernetes, but hopefully I’ve been able to demonstrate how easily a cluster can be deployed using CSE and how micro-services applications can be run in this platform.

For further reading on kubectl and all the available functionality I can recommend the Kubernetes kubectl documentation at [https://kubernetes.io/docs/reference/kubectl/overview/][24]. In fact the entire Kubernetes site is well worth a read for those considering deployment of these architectures.

As always, comments, feedback, suggestions and corrections always welcome.

Jon.

 [1]: http://152.67.105.113/?p=470
 [2]: https://vmware.github.io/container-service-extension/#tenant-installation "https://vmware.github.io/container-service-extension/#tenant-installation"
 [3]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-20.png
 [4]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image4-1.png
 [5]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image12-1.png
 [6]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image1.png
 [7]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image6.png
 [8]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image11.png
 [9]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image15.png
 [10]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image20.png
 [11]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-33.png
 [12]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-22.png
 [13]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-34.png
 [14]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-24.png
 [15]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-35.png
 [16]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-26.png
 [17]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-36.png
 [18]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-28.png
 [19]: https://github.com/microservices-demo/microservices-demo "https://github.com/microservices-demo/microservices-demo"
 [20]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-29.png
 [21]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-30.png
 [22]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-31.png
 [23]: https://kiwicloud.ninja/wp-content/uploads/2018/02/image-32.png
 [24]: https://kubernetes.io/docs/reference/kubectl/overview/ "https://kubernetes.io/docs/reference/kubectl/overview/"