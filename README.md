# Clustering and Load-Balancing

This lab intends to demonstrate how to create a simple cluster with load balancing. By the end of the lab you will have a network that consists of the gateway, the master node and the compute nodes.

## Clustering
In the first part of this tutorial, you will go through the set-up of the cluster. This requires the configuration of the servers, the installation of the **Network File System (NFS)**, the set-up of the **SSH** communication between the nodes and the **process manager** of the cluster which is included in the installation package of the **Message Passing Interface (MPI)** that will also be used for this set-up. 

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

Do not make further changes to the `hosts` file, leave the rest of the file as it is, then save the changes and exit the file.
Once you save the file, you can use the host names to `ping` or `curl` the other nodes of the cluster. For instance, check the connectivity of your network using the `ping` command on each server.

```
ping -c 5 master
ping -c 5 node1
ping -c 5 node2
```

Try these commands with different nodes on different nodes. If you have set up your network correctly, you should be able to see the information about the exchanged packages.  

In this tutorial, the master server is used as the master node of the cluster. This means that once the cluster has been set up, the _master_ node will be used to start jobs on it. These jobs will be executed by the two compute nodes, _node1_ and _node2_.   

- - - -

### Defining a user to run the Message Passing Interface (MPI) jobs

At this point, you need to create a new user for each cluster node. This new user account will be used to run the MPI jobs of each node of the cluster. Now, use the following command to create a user with username **jobhandler** and user ID **100** on every server.

```
sudo adduser jobhandler --uid 100
```

It is important that all the MPI users have the same _username_ and _user ID_ on each node for your convenience. For example, the user IDs for the MPI users need to be the same because you will give access to them on the **Network File Systems (NFS)** directory later. Permissions on the **NFS** directories are checked with respect to the user IDs. Furthermore, enter a password for the created user when prompted. It is recommended to give the same password to the MPI user of each cluster node so you have to remember only one password. The above command should also create a new home directory for the created user (i.e. `/home/jobhandler`), which you will use to execute the jobs on the cluster.

- - - -

### Install and Set up Network File System (NFS)

Files and programmes used for the execution of **MPI jobs**, which could run concurrently on the cluster, need to be available to all the nodes that participate in the execution process. Therefore, these nodes need to have access to a part of the file system on the **master** node. **NFS** enables you to mount part of a remote file system so you can access it as if it is a local directory. To install **NFS**, run the following command on the _master_ node:

```
sudo apt-get install nfs-kernel-server -y
```

In order to mount a file system on the compute nodes, the **nfs-common** package should be installed on all the compute nodes of the cluster.

``` 
sudo apt-get install nfs-common -y
```

To share the MPI user's home directory (i.e. `/home/jobhandler`) of the master node (all the MPI jobs will be run there) with the compute nodes, you need to use **NFS**. It is important that this directory is owned by the _MPI user_ of the master node so that all the _MPI users_ can access this directory. But since this home directory has been created using the **adduser** command, it is already owned by the MPI user of the master node. Use the command below to check the ownership of the directory:

```
ls -l /home/ | grep jobhandler
```

Now, share the `/home/jobhandler/` directory of the master node with all the other nodes. For this, you need to edit the file `/etc/exports` on the master node. Use the `nano` editor to add the following line to this file:

```
/home/jobhandler *(rw,sync,no_subtree_check)
```
After having finished with the installation of the **NFS** server on the _master_ node, you may need to restart it as follows:

```
sudo service nfs-kernel-server restart
```

This command also exports the directories listed in the `/etc/exports` file. In the future when this file is modified, you need to run the following command to export the directories listed in `/etc/exports` file of the _master_ node.

```
sudo exportfs -a
```

The `/home/jobhandler` directory should now be shared with the compute nodes through **NFS**. To test this, you should run the command below on any compute node getting as output the shared directory (i.e. `/home/jobhandler *`).

```
sudo showmount -e master
```
All data files and programs that will be used for running an _MPI_ job must be placed in the shared directory of the _master_ node. Then, all the other nodes, with which the master node shares the directory, will be able to access these files and programs through **NFS**.

Now, mount the `master:/home/jobhandler` from the compute nodes (i.e. _node1_ and _node2_) running the following command on each of them:

```
sudo mount master:/home/jobhandler /home/jobhandler
```

If this fails reboot the node and try again. If it works without any problem, then you should be able to see the content of the shared directory of the master node. You could also check if changes to the content of the shared directory are immediately visible to all the compute nodes by creating a new file or directory in it. If mounting the _NFS_ works, you could make it so that the `master:/home/jobhandler` directory is automatically mounted when the compute nodes are booted. To do this, use _nano_ edit the file `/etc/fstab` of each compute node adding the command line below:

```
master:/home/jobhandler /home/jobhandler nfs
```

Reboot the compute nodes and check whether the shared directory is automatically mounted by listing its contents on each compute node when you run the following command.

```
ls /home/jobhandler
```

This should show all the files and directories included in the shared directory `/home/jobhandler` of the master node.

- - - -

### Setting up the SSH communicaton between the nodes

For the cluster to work, the **master** node needs to be able to communicate securely with the compute nodes and vice versa. For this, you should use the **Secure Shell (SSH)** to establish a passwordless communication between the nodes so as to enable the _master_ node to run commands on the _compute_ nodes. This is needed to run the **MPI daemons** on the available compute nodes.

As you have already learnt in one of the previous tutorials/labs (see _Software Deployment_), to set up a SSH communication you first need to install the **SSH server** on all the nodes using the command shown below:

```
sudo apt-get install openssh-server
```
Now, you need to generate an **SSH key** for the _MPI_ users of all the cluster nodes, which will be by default created in the MPI user's home directory. Remember that in your case the _MPI_ user's home directory (i.e. `/home/jobhandler`) is actually the same directory for all nodes: the `/home/jobhandler` directory of the **master** node. Consequently, if you generate an _SSH key_ for the _MPI_ user on one of the nodes, all the other nodes will automatically have an _SSH key_. Let's generate that **SSH key**  for the _MPI_ user of the **master node** (but any node should be fine). In case that you are not logged in as `jobhandler` use the `su` command to switch to it from your current user account and then generate the _SSH key_.

```
ssh-keygen
```

When prompt for a passphrase, leave it empty. This is how you can establish a **passwordless SSH**.

After the key generation, all nodes should share the same _SSH key_. The master node needs to be able to automatically login to the compute nodes. To enable this, the _public SSH key_ of the master node needs to be added to the list of known hosts (this is usually the `authorized_keys` file that is located at `/home/jobhandler/.ssh/authorized_keys`) of all the nodes. But this is easy since all SSH key date is stored in the location `/home/jobhandler/.ssh/` on the _master_ node. Thus, instead of copying master's _public SSH key_ to all compute nodes separately, you just have to copy it to master's own `authorized_keys` file. There is a command to push the _public SSH key_ of the currently logged in user to another computer. Run the follwoing command on _master_ node as `jobhandler`:

```
ssh-copy-id localhost
```

Master's own _public SSH key_ should now be copied to `/home/jobhandler/.ssh/authorized_keys`. To check that all nodes have master's _public SSH key_ in the list of known hosts try to remotely login on the compute nodes from the master node without providing password. Finally, check that you can be securely connected to all the compute nodes using _ssh_. For example, run `ssh node1` and `ssh node2`.

If connected successfully you could `echo` the hostname of the compute node using the following command:

```
echo $HOSTNAME
```

To exit the _SSH_ communication type in `exit` in your terminal or command line.

- - - -

### Installing and Setting up the Process Manager

In this section, you go through the installation of the **process manager**, which is needed to distribute and manage the jobs that run on the cluster. The _process manager_ that will be used in this tutorial is included in the **MPICH** package, so you should start by installing that package on **all the nodes of the cluster** (preferably after updating the repositories). But, before you do this, check the available version of the **MPICH** package as follows:

```
sudo apt-cache policy mpich
```

Runnig the above command, you should get an output similar to that in the figure below.

<img src = "images/MPICH_version.png" width = "500" height = "350">

Now proceed with the update of the repositories and the installation of the **MPICH** package.

```
sudo apt-get update
sudo apt-get install mpich
```
