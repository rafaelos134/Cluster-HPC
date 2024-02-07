# Cluster-HPC

This is a *FILE TO HELP* CONFIGURE CLUSTER NODES AND MASTER


link references:
# Montando um cluster
https://www.youtube.com/watch?v=zAL4TDa5EFI

# Configurando a Rede do Cluster
https://www.youtube.com/watch?v=v3AA9V4Gqs4

# Configurando o Cluster (compartilhamento e MPI)
https://www.youtube.com/watch?v=abWiUA5mrzw

# Teste do Cluster
https://www.youtube.com/watch?v=F6sOwVTqZkM


___________________________________________________________________________

# Do this in every node and also for the main desktop 

```
sudo apt update -y && sudo apt upgrade
```

```
sudo apt install openssh-server 
```

```
sudo systemctl enable ssh
```

```
sudo systemctl status ssh
```

----------------- Configure Network ------------------------------
# Check interface of a individual computer 
# Do this for every computer that will compose the cluster
ifconfig

# It will appear the network interface data, for example
# enp0s8

# Example on how to configure the main desktop
USER@main_desktop 
sudo nano /etc/network/interfaces

# set static ip inside the editable file
# example:
#	auto enp0s8iface enp0s8 inet static   ->> set static IP
#	address 192.168.1.XXX		->> set the ip of the machine
#	netmask 255.255.255.0
#	network 192.168.1.0		->> Network address from router
#	broadcast 192.168.1.255

# ctrl+o >> ctrl+x to save and exit

# we need to do the process above for every node
# and also for the main desktop machine

# install net tools (for every computer)
sudo apt-get install net-tools

# install network interface controller
sudo apt-get install ifupdown

# Start the network
sudo ifup enp0s8

# Test the newly configured network interface
# Remember to check if everything is properly configured
ifconfig

# test other machines using ping
ping 192.168.1.xxx

# if properly configured, it will ping the selected
# machine in the network

# Now we need to configure
## DO THIS IN EVERY MACHINE
sudo nano /etc/hosts

# in the opened file, we can edit the machine names, for example
# 192.168.1.XXX main_desktop
# 192.168.1.xxx node_name_1
# 192.168.1.yyy node_name_2

# Test if it works, if it works, should ping
# example
USER@main_desktop 
ping node_name_1

__________________________________________________________________
---------------- Do this in the main desktop ---------------------

# Generating the encrypted key
USER@main_desktop 
ssh-keygen

# will output in terminal the following:
# Generating public/private rsa key pair
# Enter file in which to save the key (/home/USER/.ssh/id_rsa):

# Press enter and will appear an option to enter a passphrase
# After input the password, will generate the SHA256 key with random art inside the folder

# Now, we need to copy the key into every node
USER@main_desktop 
ssh-copy-id -i ~/home/USER/.ssh/id_rsa.pub USER@node_name

#insert password from node and its done. ***Remember to do this for every node

---------------- Do this in every node ---------------------------
# Check in every node if creation was successful
USER@node_name 
cd .ssh

USER@node_name 
ls

# after ls command should appear "authorized_keys"

---------------- Main Desktop options ----------------------------
# To enter a node from the main desktop:
USER@main_desktop 
ssh node_name

# To exit a node from the main desktop:
USER@main_desktop 
ssh exit

---------------- Parallel computing -------------------------------
# instal openmpi ** Do this for every node and also the main desktop
sudo apt-get install libopenmpi-dev

# install kernel server ** Do this in the main desktop
USER@main_desktop
sudo apt-get install nfs-kernel-server nfs-common portmap

# Create a common directory for nodes and main desktop inside the network
USER@main_desktop
ls

USER@main_desktop
mkdir clusterdir

# create folder inside exports
USER@main_desktop
sudo nano /etc/exports

#/home/USER/clusterdir will be shared the directory across the entire Network
USER@main_desktop
/home/USER/clusterdir 192.168.1.X/XX(rw,nosubtree_check,async,no_root_squash)

ctrl+s to save, ctrl+x to exit

# Reboot shared network
USER@main_desktop
sudo /etc/init.d/nfs-kernel-server restart

------------ Do this for every node ---------------
#### OBS: 192.168.1.XXX is the ip from the main desktop

USER@node_name 
sudo apt-get install nfs-common portmap

# Go to main_desktop from node
USER@node_name
showmount -e 192.168.1.XXX

# go to USER directory
USER@node_name 
pwd

USER@node_name 
mkdir clusterdir

# permanently mount nodes inside server from every node
USER@node_name 
sudo mount -t nfs 192.168.1.XXX:/home/USER/clusterdir /home/USER/clusterdir 

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
