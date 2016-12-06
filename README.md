# Clustering and Load-Balancing

This lab intends to demonstrate how to create a simple cluster with load balancing. By the end of the lab you will have a network that consists of the gateway, the master node and the compute nodes.

## Building the cluster

The cluster that will be created should consist of the following nodes (i.e. servers/computers):

* gateway
* master node and
* two compute nodes

For this lab you can use the _Alpine_ **gateway** server that you have created in the previous labs. For the creation of the _master_ and _compute_ nodes, you could use the Debian Deskop and Debian Server appliances respectively (the _links_ to download these appliances can be found at the _Cryptography_ section). 

### Creating the master node

To create the _master_ node you need to import the _Debian Desktop_ appliance, name it as **Master node** and reinitialise the MAC address as shown in the figure below. 

<img src = "images/Importing Debian Desktop.png" width = "500" height = "350">


### Creating the compute nodes

To create the first node of the two compute nodes of the cluster follow the same way as for the **master** node but this time import the _Debian Server_ appliance instead of the _Debian Desktop_ one. Now, double click on the name tab, name the new server as **Cluster node1**, tick the reinitialise MAC address box and click the _Import_ button to complete the process.

<img src = "images/Importing Debian Server.png" width = "500" height = "350">

Finally, to create the second compute node you need to make a **linked clone** of the first compute node by naming it as **Cluster node2** and reinitialising the _MAC address_.  


