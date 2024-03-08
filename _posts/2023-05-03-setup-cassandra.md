---
layout: post
title:  "How to setup cassandra cluster with Ec2 instances"
categories: cassandra db
tags: cassandra 
---

![ss](/images/cassand-cover.png)


I was assigned to a task to setup a cassandra cluster to do a POC. After a lot of trial and error, reading tens of stackoverflow answers. I got it right.


When you google for an article to setup cassandra. You will most probably find the below digital ocean article.

https://www.digitalocean.com/community/tutorials/how-to-install-cassandra-and-run-a-multi-node-cluster-on-ubuntu-22-04

The article is great but it doesn't talk about configurations required to enable authentication . I will be discussing that and changes in configuration for ec2 instances.


**Accessing remote cassandra node from jumphost**

We need to install cassandra client - cqlsh to connect with cassandra DB


1. check the version of python

```
python3 --version # it should be 3.7 or above
```
if you have the the specific version go to next step else install it.

2. Install pip3

3. Once pip3 is installed use the below command to install cqlsh

```
pip3 install cqlsh
```

4. Command to connect to remote node is
```
cqlsh <ip-address-of-node> <port> 
```
in general port is 9042

`note - the ip address it the rpc_address you mentioned . If the rpc_address is 0.0.0.0 then then the ip address is the broadcast_rpc_address you gave in cassandra.yaml file.`

Now if you see there is no authentication so who ever access the cluster will have admin privileges.

**To enable authentication and authorisation**

we need to edit cassandra.yaml to enable authentication and authorisation.

1. Set authenticator property to PasswordAuthenticator:

2. Set authorizer property to CassandraAuthorizer:


3. Set password_encryption_options property:

```
password_encryption_options:
  enabled: true
  keystore: conf/.keystore
  keystore_password: cassandra
  cipher_algorithm: AES/ECB/PKCS5Padding

```

This specifies the encryption options for passwords. In this case, passwords are encrypted using the AES/ECB/PKCS5Padding algorithm and stored in a keystore.

Once this is done restart the cassandra cluster
```
systemctl restart cassandra
```

Now you should be able to connect to cassandra nodes using

```
cqlsh <ip-address> <port> -u cassandra -p cassandra
```

These are the default username and password .

`note - these are admin credentials and should be removed once new credentials are created.`

Create a super user with the following command
```
CREATE ROLE <username> WITH PASSWORD = 'mypassword' AND LOGIN = true AND SUPERUSER = true;
```
**Changes required for cassandra in Ec2 instances:**
There is a minor changes to be made in cassandra.yaml . It is to update the enpoint_snitch value . This will help the cassandra understand the network topoly and route requests efficiently.

```
endpoint_snitch: Ec2Snitch # change to this from SimpleSnitch
```
Once this is done you need to make few more changes in cassandra-env.sh
Add the below lines in cassandra-env.sh
```
JVM_OPTS="$JVM_OPTS -Djava.rmi.server.hostname=localhost"
JVM_OPTS="$JVM_OPTS -Dcassandra.ignore_rack=true"
JVM_OPTS="$JVM_OPTS -Dcassandra.ignore_dc=true"
```

If you run into error then please look at the logs and search for that error.
log path - /var/log/cassandra/
file name - system.log ( cassandra related logs)

If you ever get connection refused it means cassandra is not up and running properly even though it shows active in systemctl status cassandra. look into the logs.

Finally some stackoverflow resources that I found useful

[StackOverflowLink1](https://stackoverflow.com/questions/18712650/cassandra-what-is-the-correct-configuration-for-ec2-multi-region)
[StackOverflowLink2](https://stackoverflow.com/questions/41776345/cassandra-failed-to-connect/41807621#41807621)