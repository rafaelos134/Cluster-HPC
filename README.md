his is a *FILE TO HELP* CONFIGURE CLUSTER NODES AND MASTER

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
# First step

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

------------------------------------------------------------------------

#  Configure Network

Check interface of a individual computer and do this for every computer that will compose the cluster.

```
ifconfig
```

It will appear the network interface data, for example `enp0s8`. Example on how to configure the main desktop:

```
USER@main_desktop
```

```
sudo nano /etc/network/interfaces
```

Set static ip inside the editable file **example**:

```
#	auto enp0s8iface enp0s8 inet static   ->> set static IP
#	address 192.168.1.XXX		->> set the ip of the machine (It is recommended to use sequences like 100, 110, 120
#	netmask 255.255.255.0
#	network 192.168.1.0		->> Network address from router
#	broadcast 192.168.1.255
```
### ctrl+o >> ctrl+x     *to save the text in nano and exit the file*

 We need to do the process above for **`every node`** and also for the **`main`** desktop machine, install net tools (for every computer):
 
 ```
sudo apt install net-tools
```

Install network interface controller

```
sudo apt-get install ifupdown
```

To start the network

```
sudo ifup enp0s8
```

Test the newly configured network interface, remember to check if everything is properly configured

```
ifconfig
```

test other machines using ping

```
ping 192.168.1.xxx
```

If properly configured, it will ping the selected machine in the network. Now we need to configure: (**`DO THIS IN EVERY MACHINE`**)

```
sudo nano /etc/hosts
```

In the opened file, we can edit the machine names, for example:

```
# 192.168.1.XXX main_desktop
# 192.168.1.xxx node_name_1
# 192.168.1.yyy node_name_2
```

Test if it works, if it works, should ping, example:

```
USER@main_desktop 
ping node_name_1
```
__________________________________________________________________
