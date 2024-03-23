This is a *FILE TO HELP* CONFIGURE CLUSTER NODES AND MASTER
___________________________________________________________________________
link references:
# Crie seu próprio Cluster de Alto Desempenho! (Raspberrie)
https://linuxuniverse.com.br/linux/cluster-hpc

# Montando um cluster
https://www.youtube.com/watch?v=zAL4TDa5EFI

# Configurando a Rede do Cluster
https://www.youtube.com/watch?v=v3AA9V4Gqs4

# Configurando o Cluster (compartilhamento e MPI)
https://www.youtube.com/watch?v=abWiUA5mrzw

# Teste do Cluster
https://www.youtube.com/watch?v=F6sOwVTqZkM


___________________________________________________________________________
# First step

obs: for facilitin, make the same name of user,"USER@..." the same in (**`ALL MACHINES`**), If you have a lot of machines, you can use the **cssh** to help

 Do this in every node and also for the main desktop 

```
sudo apt update && sudo apt upgrade -y 
```

```
sudo apt install openssh-server 
```

To force the ssh start, it's possible to use:

```
sudo systemctl enable ssh
```

```
sudo systemctl status ssh
```
For control all nodes in one computer is possible user the cluster ssh

```
sudo apt install clusterssh
```

```
cssh pi@192.168.0.2 pi@192.168.0.3 pi@192.168.0.4
```
------------------------------------------------------------------------

#  Configure Network

```
ifconfig
```

It will appear the network interface data, for example `enp0s8` or `wlp2s0`. Example on how to configure the main desktop:

```
sudo nano /etc/network/interfaces
```

Set static ip inside the editable file **example**:

```
auto enp2s0
 iface enp2s0 inet static ->> set static IP
 address 192.168.1.XXX		->> set the ip of the machine (It is recommended to use  sequences like 100, 110, 120
 netmask 255.255.255.0
 network 192.168.1.0 ->> Network address from router
 broadcast 192.168.1.255
```
### ctrl+o >> ctrl+x     *to save the text in nano and exit the file*

 We need to do the process above for **`every node`** and also for the **`main`** desktop machine, install net tools (for every computer):
 
 ```
sudo apt install net-tools
```

Install network interface controller

```
sudo apt install ifupdown
```

```
sudo service NetworkManager restart
```
To start the network

```
sudo ifup wlp2s0
```

Test the newly configured network interface, remember to check if everything is properly configured

```
ifconfig
```

test other machines using ping

```
ping 192.168.1.xxxssh-keygen
```

If properly configured, it will ping the selected machine in the network. Now we need to configure: (**`DO THIS IN EVERY MACHINE`**)

```
sudo nano /etc/hosts
```

In the opened file, we can edit the machine names, for example (you can use a generic name):

```
# 192.168.1.XXX main_desktop
# 192.168.1.xxx node_name_1
# 192.168.1.yyy node_name_2
```

Test if it works, if it works, should ping, example:

```
ping node_name_1
```
___________________________________________________________________________
# Second step
Generating the encrypted key, in main machine

```
ssh-keygen
```

Press enter for everything, the key will be in the folder (/home/USER/.ssh/id_rsa). Now, we need to copy the key into every node

```
ssh-copy-id -i ~/.ssh/id_rsa.pub USER@node_name
```
insert password from node and its done.

# Check in every node if creation was successful

Go to the folde and see, if the file was created

```
cd .ssh
```
```
cd ls
```
After ls command should appear "authorized_keys"

In main machine test the ssh to nodes, if the USER name isn't the same for all machine, you need write the USER name

```
ssh USER@node_name
```

To exit a node from the main desktop:
```
ssh exit
```
# Parallel computing

instal openmpi (**`DO THIS IN EVERY MACHINE`**)

```
sudo apt-get install libopenmpi-dev
```
In the main machine, install the kernel server (this app allows to create shared folders)

```
sudo apt-get install nfs-kernel-server nfs-common portmap
```
now in the main machine, create the shared folder in the /home

```
mkdir clusterdir
```
now configure folder sharing, go to:

```
sudo nano /etc/exports
```

and type

```
/home/USER/clusterdir 192.168.1.X/24(rw,no_subtree_check,async,no_root_squash)
```

/home/USER/clusterdir will be shared the directory across the entire Network, if you want to share with a especific machine, you can use:
```
/home/USER/clusterdir no(rw,no_subtree_check,async,no_root_squash)
```
now, reboot shared network

```
sudo /etc/init.d/nfs-kernel-server restart
```
verific if works

```
sudo systemctl status nfs-kernel-server
```
Do this in every node OBS: 192.168.1.XXX is the ip from the main desktop

```
sudo apt-get install nfs-common portmap
```
```
showmount -e 192.168.1.XXX
```
the output is "clnt_create: RPC: Unable to receive", and go to USER directory and create a folder clusterdir

```
pwd
```
```
mkdir clusterdir
```
permanently mount nodes inside server from every node

```
sudo mount -t nfs 192.168.1.XXX:/home/USER/clusterdir /home/USER/clusterdir 
```

# After mount, update line to always connect
USER@node_name
sudo nano /etc/fstab

# it will open shared network file, update the file. 
192.168.1.XXX:home/USER/clusterdir /home/USER/clusterdir nfs rw,sync,hard,int 0 0

#ctrl+o -> ctrl+x to save and exit

------------ Do this inside main desktop ---------------
# create a file to test 
USER@main_desktop
nano test.txt
aeaiejiaj

#ctrl+o -> ctrl+x to save and exit
# then, go to nodes to see if it works
USER@node_name 
cd clusterdir

USER@node_name
ls
# it should show test.txt file that is inside the main desktop

---------------- CONFIGURE OPENMPI -------------------------------
# do this in the main desktop
USER@main_desktop
pwd

USER@main_desktop
sudo nano .mpi_hostfile

# it will open a file to edit, "x" is the total number of CPU units of main desktop
# "y" and "z" are the CPU units of each node available in the network
## OBS: Be careful, if a processor have 2 cores and 4 threads, it's recommended to use only
## the two physical cores, i.e., 2. Do not input 4 taking account the threads, it can
## make the cluster slower. USE ONLY THE COUNT OF PHYSICAL CORES
local slots=x 

node_name_1=y
node_name_2=z

