---
title: Cassandra SSL for vCloud Director metrics database
author: Jon Waite
type: post
date: 2018-01-24T09:28:37+00:00
url: /2018/01/cassandra-ssl-for-vcloud-director-metrics-database/
categories:
  - Cassandra
tags:
  - Cassandra
  - Configuration
  - Encryption
  - Installation
  - Metrics
  - Security
  - SSL
  - vCloud Director 9

---
In vCloud Director v9 the requirement for both [Apache Cassandra][1] and [KairosDB][2] for storing metrics has been reduced to just Apache Cassandra. In addition, the ability to view VM metrics directly in the new  vCD 9 HTML5 tenant UI makes it much more important to have a reliable Cassandra infrastructure.

As I found when researching this post, configuring SSL in Cassandra is a bit of a pain, since Cassandra runs as a Java application it has some issues with the types of CA certificates and trusts it can use, which is further complicated by the options of both node to node encryption as well as cluster to clients.

I set out to produce an 'easy' way to configure a Cassandra cluster with SSL for both node to node and node to client communication in a way which could be reasonably easily implemented and reproduced for future installs.

My test environment consists of the minimum supported configuration of 4 Cassandra nodes (of which 2 are 'seed' nodes) running on CentOS Linux 7 (since that's what we tend to use most for our backend infrastructure services). This configuration can almost certainly be adapted for other Linux distributions and I've tried to document the certificate generation process sufficiently that this will be straightforward.

Inspiration for this post was from [Antoni Spiteri's][3] blog and script to [configure a Cassandra cluster for vCloud Director metrics][4] which I found extremely useful background.

Initial setup of my node servers used the following pattern to install and configure Cassandra:

Ensure all appropriate updates have been applied to our new CentOS installation if appropriate:

`# yum update -y`

Install Java (currently on my systems this installs java-1.8.0-openjdk.x86\_64 1:1.8.0.161-b14.el7\_4 which appears to work fine):

`# yum install -y java`

Create a file `/etc/yum.repos.d/cassandra.repo` (with vi or your favourite text editor) to include the Cassandra 3.0.x repository with the following contents. Note that I'm using the Cassandra 3.0.x repository (30X) and not the latest 311x release repository as this is not yet supported by VMware for vCloud Director:

`[cassandra]`  
 `name=Apache Cassandra`  
 `baseurl=https://www.apache.org/dist/cassandra/redhat/30x/`  
 `gpgcheck=1`  
 `repo_gpgcheck=1`  
 `gpgkey=https://www.apache.org/dist/cassandra/KEYS`

Install Cassandra software itself (currently on my test nodes this pulls in the `cassandra.noarch 0:3.0.15-1` package):

`# yum install -y cassandra`

The main configuration file for Cassandra is (on CentOS installed from this repository) in /etc/cassandra/conf/cassandra.yaml.

At least the following options in this file will need to be changed before we can get a cluster up and running:

<table>
  <tr>
    <td>
      <strong>Original cassandra.yaml</strong>
    </td>
    
    <td>
      <strong> Edited cassandra.yaml</strong>
    </td>
    
    <td>
      <strong>Notes</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      cluster_name: 'Test Cluster'
    </td>
    
    <td>
      cluster_name: 'My vCD Cluster'
    </td>
    
    <td>
      Doesn't absolutely have to be changed, but you probably should do. Note that this setting must exactly match on each of your node servers or they won't be able to join the cluster.
    </td>
  </tr>
  
  <tr>
    <td>
      authenticator: AllowAllAuthenticator
    </td>
    
    <td>
      authenticator: PasswordAuthenticator
    </td>
    
    <td>
      We'll want to use password security from vCloud Director to the cluster.
    </td>
  </tr>
  
  <tr>
    <td>
      authorizer: AllowAllAuthorizer
    </td>
    
    <td>
      authorizor: CassandraAuthorizer
    </td>
    
    <td>
      Required to enforce password security.
    </td>
  </tr>
  
  <tr>
    <td>
       - seeds: &#8220;127.0.0.1&#8221;
    </td>
    
    <td>
      - seeds: &#8220;Seed Node 1 IP address,Seed Node 2 IP address&#8221;
    </td>
    
    <td>
      Set the seeds for the cluster (minimum 2 nodes must be configured as seeds).
    </td>
  </tr>
  
  <tr>
    <td>
      listen_address: localhost
    </td>
    
    <td>
      listen_address: This node IP address
    </td>
    
    <td>
      Must be configured for the node to listen for non-local traffic (from other nodes and clients).
    </td>
  </tr>
  
  <tr>
    <td>
      rpc_address: localhost
    </td>
    
    <td>
      rpc_address: This node IP address
    </td>
    
    <td>
      Must be configured for the node to listen for non-local traffic (from other nodes and clients).
    </td>
  </tr>
</table>

We'll need to change some additional settings later to implement SSL security, but these settings should be enough to get the cluster functioning.

You'll also need to permit the Cassandra traffic through the default CentOS 7 firewall, the following commands will open the appropriate ports (as root):

`# firewall-cmd --zone public --add-port 7000/tcp --add-port 7001/tcp --add-port 7199/tcp --add-port 9042/tcp --add-port 9160/tcp --add-port 9142/tcp --permanent`  
`# firewall-cmd --reload`

Once you've performed these steps on each of the 4 nodes, you should be able to bring up a (non-encrypted) cassandra cluster by running:

`# service cassandra start`

on each node, you should probably also enable the service to auto-start on server reboot:

`# chkconfig cassandra on`

Note that you should start the nodes 1 by 1 and allow a minimum of 30 seconds between each one to allow the cluster to reconfigure as each node is added before adding the next one, check `/var/log/cassandra/cassandra.log` and `/var/log/cassandra/system.log` if you have issues with the cluster forming.

My test cluster has 4 nodes (named node01, node02, node03 and node04 imaginatively enough), and node01 and node02 are the seeds. The IP addresses are 10.0.0.101,102,103 and 104 (/24 netmask).

If everything has worked ok running 'nodetool status' on any node should show the cluster members all in a state of 'UN' (Up/Normal):

[<img loading="lazy" decoding="async" class="aligncenter size-large wp-image-433" src="https://kiwicloud.ninja/wp-content/uploads/2018/01/nodetool_status-800x162.png" alt="" width="800" height="162" srcset="https://kiwicloud.ninja/wp-content/uploads/2018/01/nodetool_status-800x162.png 800w, https://kiwicloud.ninja/wp-content/uploads/2018/01/nodetool_status-300x61.png 300w, https://kiwicloud.ninja/wp-content/uploads/2018/01/nodetool_status-768x156.png 768w, https://kiwicloud.ninja/wp-content/uploads/2018/01/nodetool_status-150x30.png 150w, https://kiwicloud.ninja/wp-content/uploads/2018/01/nodetool_status-250x51.png 250w, https://kiwicloud.ninja/wp-content/uploads/2018/01/nodetool_status.png 818w" sizes="(max-width: 800px) 100vw, 800px" />][5]

You should also be able to login to the cluster via any of the nodes using the cqlsh command (the default password is 'cassandra' for the builtin cassandra user)

`[root@node01 ~]# cqlsh 10.0.0.101 -u cassandra -p cassandra<br />
Connected to My vCD Cluster at 10.0.0.101:9042.<br />
[cqlsh 5.0.1 | Cassandra 3.0.15 | CQL spec 3.4.0 | Native protocol v4]<br />
Use HELP for help.<br />
cassandra@cqlsh>`

To reconfigure the cluster for SSL encrypted communication we need to complete a number of tasks:

  * Create a new CA certificate authority (we could use an existing / external CA authority, but I'm trying to keep this simple).
  * Export the public key for our new CA so we can tell vCloud Director later to trust certificates it has issued.
  * Create a new truststore for Cassandra so Cassandra trusts our CA.
  * Create a new keystore for Cassandra for each node which includes the public key of our CA.
  * Create a certificate request from each node and submit this to our CA for signing.
  * Import the certificate response from the CA into the keystore for each node.
  * Move the generated truststore and keystore into appropriate locations and configure security on the files.
  * Reconfigure Cassandra to enable encryption and use our certificates.

That's a lot of steps to go through and extremely tedious to get right, so I wrote a script to do most of these steps, create a file (I called my 'gencasscerts.sh' on one of the node servers and copy/paste the script contents from below):

<pre class="theme:github lang:sh decode:true">#!/bin/bash

NODENAMES=(node01 node02 node03 node04)

STORETYPE="PKCS12"
CASTOREPASS="capassword"
NODESTOREPASS="nodepassword"
CLUSTERCN="cassandra"
CERTOPTS="OU=MyDept,O=MyOrg,L=MyCity,S=MyState,C=US"

# Generate our CA certificate
/usr/bin/keytool -genkey -keyalg RSA -keysize 2048 -alias myca \
-keystore myca.p12 -storepass ${CASTOREPASS} -storetype ${STORETYPE} \
-validity 3650 -dname "CN=${CLUSTERCN},${CERTOPTS}"

# Export the public key of our new CA to myca.pem:
/usr/bin/openssl pkcs12 -in myca.p12 -nokeys -out myca.pem -passin pass:${CASTOREPASS}

# Export the private key of our new CA to myca.key:
/usr/bin/openssl pkcs12 -in myca.p12 -nodes -nocerts -out myca.key -passin pass:${CASTOREPASS}

# Create a truststore that only includes the public key of our CA
/usr/bin/keytool -importcert -keystore .truststore -storepass ${CASTOREPASS} -storetype ${STORETYPE} -file myca.pem -noprompt

((n_elements=${#NODENAMES[@]}, max_index=n_elements-1))
for ((i = 0; i <= max_index; i++)); do
curnode=${NODENAMES[i]}
echo "Processing node $i: ${curnode}"

# Generate node certificate:
/usr/bin/keytool -genkey \
-keystore ${curnode}.p12 -storepass ${NODESTOREPASS} -storetype ${STORETYPE} \
-keyalg RSA -keysize 2048 -validity 3650 \
-alias ${curnode} -dname "CN=${curnode},${CERTOPTS}"

# Import the root CA public cert to our node cert:
/usr/bin/keytool -import -trustcacerts -noprompt -alias myca \
-keystore ${curnode}.p12 -storepass ${NODESTOREPASS} -file myca.pem

# Generate node CSR:
/usr/bin/keytool -certreq -alias ${curnode} -keyalg RSA -keysize 2048 \
-keystore ${curnode}.p12 -storepass ${NODESTOREPASS} -storetype ${STORETYPE} \
-file ${curnode}.csr

# Sign the node CSR from our CA:
/usr/bin/openssl x509 -req -CA myca.pem -CAkey myca.key -in ${curnode}.csr -out ${curnode}.crt -days 3650 -CAcreateserial

# Import the signed certificate into our node keystore:
/usr/bin/keytool -import -keystore ${curnode}.p12 -storepass ${NODESTOREPASS} -storetype ${STORETYPE} \
-file ${curnode}.crt -alias ${curnode}

mkdir ${curnode}
cp .truststore ${curnode}/.truststore
cp ${curnode}.p12 ${curnode}/.keystore
cp ${curnode}.crt ${curnode}/client.pem

done
</pre>

**Update 2018/01/26: I realised shortly after publishing that the original version of this script included both the public and private keys for the CA in the .truststore keystore. While this isn't a huge issue in a private environment it's definitely not 'best practice' to distribute the private key of the CA to each node so I've refined the script and removed the private keys from the generated .truststore files in this version. If you need to regenerate your environment keys remember that if you re-run the script both the .truststore and .keystore files will need to be updated on each node.**

Make the script executable using:

`chmod u+x gencasscerts.sh`

Edit the settings and passwords for the certificates (we'll need these later) in the variables at the top of the file to be appropriate for your environment (in particular the node names in the 3rd line) and any other options you want to change. The default validity period of the generated certificates is set to 10 years (3650 days). Obviously you should also change the passwords in your copy of the script for the CASTOREPASS and NODESTOREPASS variables.

When you run the script you should get output similar to the following:  
`[root@node01 ~]# ./gencasscerts.sh<br />
MAC verified OK<br />
MAC verified OK<br />
Processing node 0: node01<br />
Certificate was added to keystore<br />
Signature ok<br />
subject=/C=US/ST=MyState/L=MyCity/O=MyOrg/OU=MyDept/CN=node01<br />
Getting CA Private Key<br />
Certificate reply was installed in keystore<br />
Processing node 1: node02<br />
Certificate was added to keystore<br />
Signature ok<br />
subject=/C=US/ST=MyState/L=MyCity/O=MyOrg/OU=MyDept/CN=node02<br />
Getting CA Private Key<br />
Certificate reply was installed in keystore<br />
Processing node 2: node03<br />
Certificate was added to keystore<br />
Signature ok<br />
subject=/C=US/ST=MyState/L=MyCity/O=MyOrg/OU=MyDept/CN=node03<br />
Getting CA Private Key<br />
Certificate reply was installed in keystore<br />
Processing node 3: node04<br />
Certificate was added to keystore<br />
Signature ok<br />
subject=/C=US/ST=MyState/L=MyCity/O=MyOrg/OU=MyDept/CN=node04<br />
Getting CA Private Key<br />
Certificate reply was installed in keystore`

Looking at the directory from where the script was run you should see the certificate files:

[<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-443" src="https://kiwicloud.ninja/wp-content/uploads/2018/01/gencerts_files.png" alt="" width="470" height="336" srcset="https://kiwicloud.ninja/wp-content/uploads/2018/01/gencerts_files.png 470w, https://kiwicloud.ninja/wp-content/uploads/2018/01/gencerts_files-300x214.png 300w, https://kiwicloud.ninja/wp-content/uploads/2018/01/gencerts_files-150x107.png 150w, https://kiwicloud.ninja/wp-content/uploads/2018/01/gencerts_files-210x150.png 210w" sizes="(max-width: 470px) 100vw, 470px" />][6]

In each node directory there will be 3 files (.keystore, .truststore and chain.pem):

[<img loading="lazy" decoding="async" class="aligncenter size-full wp-image-444" src="https://kiwicloud.ninja/wp-content/uploads/2018/01/nodefiles.png" alt="" width="433" height="113" srcset="https://kiwicloud.ninja/wp-content/uploads/2018/01/nodefiles.png 433w, https://kiwicloud.ninja/wp-content/uploads/2018/01/nodefiles-300x78.png 300w, https://kiwicloud.ninja/wp-content/uploads/2018/01/nodefiles-150x39.png 150w, https://kiwicloud.ninja/wp-content/uploads/2018/01/nodefiles-250x65.png 250w" sizes="(max-width: 433px) 100vw, 433px" />][7]

The .truststore files will all be identical between the node directories, the .keystore and client.pem files will be unique.

Next we need to move the generated certificate stores to an appropriate location, easiest way to do this is to use scp from the directory where the script was run. We'll place the files in the Cassandra configuration directory (/etc/cassandra/conf).

On the node where the files have been generated we can just copy them (node01 in our case):

`# cp node01/.keystore node01/.truststore /etc/cassandra/conf`

To copy the appropriate files to the other nodes we can use scp:

`# scp node02/.keystore node02/.truststore root@10.0.0.102:/etc/cassandra/conf<br />
# scp node03/.keystore node03/.truststore root@10.0.0.103:/etc/cassandra/conf<br />
# scp node04/.keystore node04/.truststore root@10.0.0.104:/etc/cassandra/conf`

On each node we now run the following to set appropriate permissions and ownership on the files:

`# chown cassandra:cassandra /etc/cassandra/conf/.keystore /etc/cassandra/conf/.truststore<br />
# chmod 400 /etc/cassandra/conf/.keystore /etc/cassandra/conf/.truststore`

We can now reconfigure the `cassandra.yaml` configuration file on each node to use our new certificates and enable encrypted communication.

The original settings from `cassandra.yaml` and the required changes are:

In the `server_encryption_options:` section to encrypt node-to-node communications:

<table>
  <tr>
    <td>
      <strong>Original Value</strong>
    </td>
    
    <td>
      <strong>New Value</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      internode_encryption: none
    </td>
    
    <td>
      internode_encryption: all
    </td>
  </tr>
  
  <tr>
    <td>
      keystore: conf/.keystore
    </td>
    
    <td>
      keystore: <location of copied .keystore file>
    </td>
  </tr>
  
  <tr>
    <td>
      keystore_password: cassandra
    </td>
    
    <td>
      keystore_password: <value you set for the NODESTOREPASS variable in the script above>
    </td>
  </tr>
  
  <tr>
    <td>
      truststore: conf/.truststore
    </td>
    
    <td>
      truststore: <location of copied .truststore file>
    </td>
  </tr>
  
  <tr>
    <td>
      truststore_password: cassandra
    </td>
    
    <td>
      truststore_password: <value you set for the CASTOREPASS variable in the script above>
    </td>
  </tr>
</table>

&nbsp;

Optionally you can also change the `cipher_suites:` setting to restrict available ciphers to the more secure versions (e.g. `cipher_suites: [TLS_RSA_WITH_AES_256_CBC_SHA]'`).

In the `client_encryption_options:` section to encrypt node-to-client communications:

<table>
  <tr>
    <td>
      <strong>Original Value</strong>
    </td>
    
    <td>
      <strong>New Value</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      enabled: false
    </td>
    
    <td>
      enabled: true
    </td>
  </tr>
  
  <tr>
    <td>
      keystore: conf/.keystore
    </td>
    
    <td>
      keystore: <location of copied .keystore file>
    </td>
  </tr>
  
  <tr>
    <td>
      keystore_password: cassandra
    </td>
    
    <td>
      keystore_password: <value you set for the NODESTOREPASS variable in the script above>
    </td>
  </tr>
</table>

Again you can change the `cipher_suites:` setting if desired to use more secure ciphers.

Now we need to stop the cassandra service on ALL nodes:

`service cassandra stop`

Starting the cassandra service back up (`service cassandra start` - remember to wait between each node to give the cluster time to settle) you should now see the following in the `/var/log/cassandra/system.log` file:

`INFO [main] 2018-01-25 20:52:38,290 MessagingService.java:541 - Starting Encrypted Messaging Service on SSL port 7001`

Once the nodes are all back up and running `nodetool status` should show them all as status of 'UN' (Up/Normal).

If we attempt to use cqlsh to connect to the cluster now, we should get an error as we're not using an encrypted connection:

`cqlsh 10.0.0.101 -u cassandra -p cassandra<br />
Connection error: ('Unable to connect to any servers', {'10.0.0.101': ConnectionShutdown('Connection <AsyncoreConnection(24713424) 10.0.0.101:9042 (closed)> is already closed',)})`

If we specify the '`--ssl`' flag to cqlsh, we still get an error as we haven't provided a client certificate for the connection:

`cqlsh 10.0.0.101 -u cassandra -p cassandra --ssl<br />
Validation is enabled; SSL transport factory requires a valid certfile to be specified. Please provide path to the certfile in [ssl] section as 'certfile' option in /root/.cassandra/cqlshrc (or use [certfiles] section) or set SSL_CERTFILE environment variable.`

This is where the client.pem file is used as generated by the script, copy this file into the .cassandra folder in your user home path and then create/edit a file in this .cassandra folder called cqlshrc with the following content:

`[connection]<br />
factory = cqlshlib.ssl.ssl_transport_factory<br />
[ssl]<br />
certfile = ~/.cassandra/client.pem<br />
validate = false`

Save the file and now we should be able to establish an encrypted session:

`cqlsh 10.0.0.101 -u cassandra -p cassandra --ssl<br />
Connected to My vCD Cluster at 10.0.0.101:9042.<br />
[cqlsh 5.0.1 | Cassandra 3.0.15 | CQL spec 3.4.0 | Native protocol v4]<br />
Use HELP for help.<br />
cassandra@cqlsh>`

When configuring vCloud Director to use our new metrics cluster, we must first tell vCloud Director that it can trust the CA we've created for our Cassandra cluster by importing the public key of our CA (the myca.pem file generated by the script) into the vCloud Director cell server cacerts repository. Ludovic Rivallain has a great post written up at <https://vuptime.io/2017/08/30/VMware-Patch-vCloudDirector-cacerts-file/> showing how to do this. Note that this must be performed on each vCloud Director cell server as the cacerts repository is not shared between them.

You should also add a new admin user to Cassandra with a complex password and disable the builtin 'cassandra' user account before using the cluster.

Finally, you can follow the VMware documentation ([link][8]) to configure your vCloud Director cell to use this Cassandra cluster for metrics storage.

It's reasonably easy to adjust this process to use an external CA rather than generating a new self-signing one, but this post is long enough already so let me know in the comments if you'd like to see this and I'll write up a separate post detailing the changes to do this.

As always, comments/corrections/feedback welcome.

Jon.

 [1]: http://cassandra.apache.org/
 [2]: https://kairosdb.github.io/
 [3]: https://anthonyspiteri.net/
 [4]: https://anthonyspiteri.net/configuring-cassandra-for-vcloud-director-9-0-metrics/
 [5]: https://kiwicloud.ninja/wp-content/uploads/2018/01/nodetool_status.png
 [6]: https://kiwicloud.ninja/wp-content/uploads/2018/01/gencerts_files.png
 [7]: https://kiwicloud.ninja/wp-content/uploads/2018/01/nodefiles.png
 [8]: https://docs.vmware.com/en/vCloud-Director/9.0/com.vmware.vcloud.install.doc/GUID-E5B8EE30-5C99-4609-B92A-B7FAEC1035CE.html