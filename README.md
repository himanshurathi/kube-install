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

## Verify that the br_netfilter, overlay modules are loaded by running the following commands, on both VMs :

    lsmod | grep br_netfilter
    lsmod | grep overlay

## Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command, on both VMs :

    sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

## 8. Install the run time, cri-o on both VMs.

    OS="xUbuntu_22.04"

    VERSION="1.28"

    cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list

    deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /

    EOF

    cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list

    deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /

    EOF

    curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -

    curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -

    sudo apt-get update

    sudo apt-get install cri-o cri-o-runc cri-tools -y

    sudo systemctl daemon-reload

    sudo systemctl enable crio --now

    service crio status

## 9. Set up kubeadm, kubelet and kubectl.

## On Both VMs --

    sudo apt-get update

    sudo apt-get install -y apt-transport-https ca-certificates curl gpg

    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt-get update

## On kubem --
    sudo apt-get install -y kubelet kubeadm kubectl

    sudo apt-mark hold kubelet kubeadm kubectl

## On kuben --

    sudo apt-get install -y kubelet kubeadm

    sudo apt-mark hold kubelet kubeadm

## 10. Now finally to initialise kubernetes master run the below command after replacing the kubemip with kubernetes master ip :

    kubeadm init --apiserver-advertise-address=kubemip --apiserver-cert-extra-sans=kubemip --pod-network-cidr=192.168.0.0/16 --node-name kubem

## Ensure to the note the join command displayed at the end, this is what you need to run on the kuben to have that join the cluster.

## After the above step, your kubernetes master components starts coming up and should be up and running but to interact with kubernetes master you need a kubeconfig file which helps you authenticate/authorise with the cluster.

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

## Now your cluster is up and running and you can communicate with it using kubectl command line.

    kubectl get nodes

## 11. Enable calico plugin on master →
 
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
    
    curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml -O

    kubectl create -f custom-resources.yaml

    kubectl get pods -n calico-system