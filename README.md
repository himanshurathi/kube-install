# This Documentation is to install kubernetes using kubeadm. We will be using kubernetes master version 1.29 for the ubuntu-22.04.

##1. Connect with lab and provision the two vms and connect with
them. Vm-2 – kubem, vm-3 kuben


##2. On kubernetes master
    sudo -i
    hostnamectl set-hostname kubem
    exec bash

   On kubernetes node
    sudo -i
    hostnamectl set-hostname kuben
    exec bash


##3. On both the vms.
    apt install net-tools
    ifconfig


##4. Now we are going to fix the communicate between two nodes.

On kubem →
    nano /etc/hosts 
    root@kubem:~# cat /etc/hosts
    172.31.88.43 kubem
    127.0.0.1 kubem localhost
    172.31.88.77 kuben


On kuben →
    root@kuben:~# cat /etc/hosts
    172.31.88.77 kuben
    127.0.0.1 kuben localhost
    172.31.88.43 kubem



5. Ping each other and verify they are
reachable on hostnames.
ping kuben - from master
ping kubem – from node
