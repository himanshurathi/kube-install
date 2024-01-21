# This Documentation is to install kubernetes using kubeadm. We will be using kubernetes master version 1.29 for the ubuntu-22.04.

## 1. Connect with lab and provision the two vms and connect with them. We will be treating Vm-2 – kubem (as kubernetes master), vm-3 - kuben (as kubernetes node)

## 2.1 On kubernetes master

    sudo -i
    hostnamectl set-hostname kubem
    exec bash

## 2.2 On kubernetes node

    sudo -i
    hostnamectl set-hostname kuben
    exec bash

## 3. On both the vms.

    apt install net-tools
    ifconfig

## 4. Now we are going to fix the communication between two nodes.

## On kubem →

    nano /etc/hosts
    root@kubem:~# cat /etc/hosts
    172.31.88.43 kubem
    127.0.0.1 kubem localhost
    172.31.88.77 kuben

## On kuben →

    root@kuben:~# cat /etc/hosts
    172.31.88.77 kuben
    127.0.0.1 kuben localhost
    172.31.88.43 kubem

## 5. Ping each other and verify they can reach other on hostnames.

## from kubem

        ping kuben

## from kuben

        ping kubem

## Both must be pinging each other.

## 6. In a cluster swap memory must be disabled on all the participating machines.

## On both the VMs

    swapoff -a

## If swap entries are in /etc/fstab – comment them and restart the VM.

## 7. Fix the IPtables to work with bridge networking.

## On both the VMs

    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

## To load these modules, On both VMs

    sudo modprobe overlay
    sudo modprobe br_netfilter

## sysctl params required by setup, params persist across reboots. On both VMs

    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

## Apply sysctl params without reboot. On both VMs

    sudo sysctl --system
