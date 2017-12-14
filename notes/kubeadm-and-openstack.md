# How to use kubeadm with openstack

Kubernetes is an open-source platform designed to automate deploying, scaling, and operating application containers.
It eliminates many of the manual processes involved in deploying and scaling containerized applications.

In order for us to use kubernetes, we have to install and run it some where like an operating system or cloud. The default installer
for kubernetes is kubeadm. 

In this post, I will describe deployment of kubernets 1.9 to openstack using kubeadm. I will cover

+ Creating network instances on openstack to host kubernetes
+ Install kubeadm and docker
+ Work with kubeadm
    
## 1.Create Nodes on openstack

Because kubernetes needs to be run somewhere, we shall create three network instances or nodes on openstack. 

    openstack server create --image xenial --key-name curtis --flavor m1.medium --min 3 --max 3 --nic net-id=cee24724-e062-4370-ba9f-57bed80f32cd \kubernetes

We should have one master and two worker nodes. You can see the created kubernetes nodes.

    os server list

## 2.Install kubeadm and docker on the Nodes

### 1.install on master Node

<br> 1.ssh into the master node. </br>
<br> 2.create a script and name it install.h with the contents below. </br>

    apt-get update
    apt-get install apt-transport-https ca-certificates curl software-properties-common
    apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    apt-get update
    apt-get install docker-ce -y --allow-unauthenticated
    apt-get install -y kubeadm kubectl kubernetes-cni --allow-unauthenticated
    kubeadm init --skip-preflight-checks
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    kubectl --kubeconfig .kube/config apply -f https://git.io/weave-kube-1.6
 
 3.Run the script.
 
    sudo sh install.sh
    
This installs docker and kubernetes packages.

### 2.Install on worker Nodes

<br> 1.ssh into the each worker node. </br>
<br> 2.Run the same install script we used on master. </br>
 
    sudo sh install.sh

## 3.Work with kubeadm

### Initialize Kubernetes master

We can now use kubeadm to initialize kubernetes on the master Node.

    sudo kubeadm init
    
### Join worker nodes

Also use kubeadm join to add workers.

    sudo   kubeadm join --token bdc910.dac015f93ad5a064 10.50.0.11:6443
    
If this ran successfully, then you should be able to see the nodes.

    kubectl --kubeconfig ./admin.conf get nodes



