# Clustering and Load-Balancing

This lab intends to demonstrate how to create a simple cluster with load balancing. By the end of the lab you will have a network that consists of the gateway, the master node and the compute nodes.

## Clustering
In the first part of this tutorial, you will go through the set-up of the cluster. 

### Building the cluster

The cluster that will be created should consist of the following nodes (i.e. servers/computers):

* gateway
* master node and
* two compute nodes

For this lab you can use the _Alpine_ **gateway** server that you have created in the previous labs. For the creation of the _master_ and _compute_ nodes, you could use the Debian Deskop and Debian Server appliances respectively (the _links_ to download these appliances can be found at the _Cryptography_ section). 

#### Creating the Master Node

To create the _master_ node you need to import the _Debian Desktop_ appliance, name it as **Master node** and reinitialise the MAC address as shown in the figure below. 

<img src = "images/Importing Debian Desktop.png" width = "500" height = "350">


#### Creating the Compute Nodes

To create the first node of the two compute nodes of the cluster follow the same way as for the **master** node but this time import the _Debian Server_ appliance instead of the _Debian Desktop_ one. Then, double click on the name tab, name the new server as **Cluster node1**, tick the reinitialise MAC address box and click the _Import_ button to complete the process.

<img src = "images/Importing Debian Server.png" width = "500" height = "350">

Finally, to create the second compute node you need to make a **linked clone** of the first compute node by naming it as **Cluster node2** and reinitialising the _MAC address_.  

**NOTE:** Before you boot all your cluster nodes up, check that their network adapters are all set to the **Internal Network** configuration.

- - - -

### Configuring the Cluster Nodes

Boot up your **gateway** server and make sure that **DHCP** services are running. After that, boot up all the other servers and log in either as `root` or `newuser` (preferably as newuser).

* To log in to the _compute nodes_, you need to enter the following credentials:
 * `root` or `newuser` as username and
 * `raspberry` as password.
 
* To log in to the _master node_, you need to user `newuser` as username and `raspberry` as password.

Once you have logged in to all the servers, find their IP addressess and keep a note of them (In this case, the IP addresses are: `10.5.5.10` for the _master_ node, `10.5.5.9` for _node1_ and `10.5.5.18` for _node2_). Furthermore, change the name of the cluster nodes by using `nano` to edit their `hostname` file and substitute `debian` for the name of each node. For example, the new hostnames of the master node and the two compute nodes could be `master`, `node1` and `node2` respectively. 

```
sudo nano /etc/hostname
```

You also need to edit the `hosts` file of each node (i.e. master, node1 and node2) using `sudo nano /etc/hosts`. Then, add the following lines after the first two lines of the file. 

```
10.5.5.10 master
10.5.5.9  node1
10.5.5.18 node2
```

Do not make further changes to the `host` file, leave the rest of the file as it is, then save the changes and exit the file.
Once you save the file, you can use the host names to `ping` or `curl` the other nodes of the cluster. For instance, check the connectivity of your network using the `ping` command on each server.

```
ping -c 5 master
ping -c 5 node1
ping -c 5 node2
```

Try these commands with different nodes on different nodes. If you have set up your network correctly, you should be able to see the information about the exchanged packages.  

In this tutorial, the master server is used as the master node of the cluster. This means that once the cluster has been set up, the _master_ node will be used to start jobs on it. These jobs will be executed by the two compute nodes, _node1_ and _node2_.   

- - - -







