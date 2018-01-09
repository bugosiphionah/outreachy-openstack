# How to use kubeadm with openstack using kubernetes 1.9

Kubernetes is an open-source platform designed to automate deploying, scaling, and operating application containers.
It eliminates many of the manual processes involved in deploying and scaling containerized applications.

In order for us to use kubernetes, we have to install and run it some where like an operating system or cloud. In this case we will run 3 VM(s) in an OpenStack environment as we follow the steps in the blog. You can run atleast 2 VM(s) so that one acts as the master and the other as the node.

The default installer for kubernetes is kubeadm and in this post, I will describe deployment of kubernets 1.9 to openstack using kubeadm. I will cover

+ Creating network instances on openstack to host kubernetes
+ Installing kubeadm and docker
+ Working with kubeadm

## Requirements
To follow along seamlessly, you should have the following in place:
+ Setup openstack. There is good [documentation](https://www.openstack.org/software/start/) to guide you.
+ Also the steps will work assuming debian based environments.
    
## 1.Create Nodes on openstack

As empahased earlier on, because kubernetes needs to be run somewhere, we shall create three network instances or nodes on openstack. 

    openstack server create --image xenial --key-name curtis --flavor m1.medium --min 3 --max 3 --nic net-id=cee24724-e062-4370-ba9f-57bed80f32cd \kubernetes

We should have one master and two worker nodes. You can see the created kubernetes nodes.

    os server list

## 2.Install kubernetes and docker packages on the Nodes

### 1.Install on master Node

<br> 1.ssh into the master node. </br>
<br> 2.create a script and name it master.sh with the contents below. </br>
    
    #Install dependencies
    apt-get update -y && apt-get upgrade -y
    apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
    add-apt-repository ppa:gophers/archive
    apt-get install golang-1.9-go
   
    #install docker
    apt-get update
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
       $(lsb_release -cs) \
       stable"
    apt-get update -y && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.09 | head -1 | awk '{print         $3}')
    # patch up docker so cgroup driver used by kubelet is the same as the one used by Docker
    cat << EOF > /etc/docker/daemon.json
    {
      "exec-opts": ["native.cgroupdriver=cgroupfs"]
    }
    EOF
    service docker restart
    # cleanup any old images, containers in docker
    docker rm -f $(docker ps -a -q)
    docker rmi -f $(docker images -q -a)
    
    # install latest kubernetes binaries
    apt-get update
    apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main
    apt-get install -y kubelet kubeadm kubectl kubernetes-cni --allow-unauthenticated
    
    kubeadm init --skip-preflight-checks
    cat << EOF > kubeadm.conf
    kind: MasterConfiguration
    apiVersion: kubeadm.k8s.io/v1alpha1
    cloudProvider: openstack
    kubernetesVersion: 1.9.0
    unifiedControlPlaneImage: "gcr.io/google_containers/hyperkube-amd64:v1.9.0"
    EOF
    echo "token: "$(kubeadm token generate) >> kubeadm.conf
    
    # Fix /etc/hosts to add entries from cloud.conf on BOTH master and node
    # Copy cloud.conf over to the VM(s) in /etc/kubernetes/cloud-config on BOTH master and node

    # patch /etc/systemd/system/kubelet.service.d/10-kubeadm.conf on both VM(s) to add the cloud provider related arguments
    sed -i -E 's/(.*)KUBELET_KUBECONFIG_ARGS=(.*)$/\1KUBELET_KUBECONFIG_ARGS=--cloud-provider=openstack --cloud-            config=\/etc\/kubernetes\/cloud-config \2/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

    systemctl daemon-reload
    systemctl restart kubelet
    
    mkdir -p $HOME/.kube
    sudo rm $HOME/.kube/config
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chmod 755 $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    export kubever=$(kubectl version | base64 | tr -d '\n')
    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"

    kubectl create -f - <<EOF
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: standard
      annotations:
        storageclass.beta.kubernetes.io/is-default-class: "true"
      labels:
        kubernetes.io/cluster-service: "true"
        addonmanager.kubernetes.io/mode: EnsureExists
    provisioner: kubernetes.io/cinder
 
 3.Run the script.
 
    sudo sh install.sh
    
This installs docker, kubernetes packages and other dependencies..

### 2. Install on worker Nodes

<br> 1.ssh into the each worker node. </br>
<br>2.2.create a script and name it worker.sh with the contents below.  </br>

    #Install dependencies
    apt-get update -y && apt-get upgrade -y
    apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
    add-apt-repository ppa:gophers/archive
    apt-get install golang-1.9-go
   
    #install docker
    apt-get update
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
       $(lsb_release -cs) \
       stable"
    apt-get update -y && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.09 | head -1 | awk '{print         $3}')
    # patch up docker so cgroup driver used by kubelet is the same as the one used by Docker
    cat << EOF > /etc/docker/daemon.json
    {
      "exec-opts": ["native.cgroupdriver=cgroupfs"]
    }
    EOF
    service docker restart
    # cleanup any old images, containers in docker
    docker rm -f $(docker ps -a -q)
    docker rmi -f $(docker images -q -a)
    
    # install latest kubernetes binaries
    apt-get update
    apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main
    apt-get install -y kubelet kubeadm kubectl kubernetes-cni --allow-unauthenticated
    
    # Fix /etc/hosts to add entries from cloud.conf on BOTH master and node
    # Copy cloud.conf over to the VM(s) in /etc/kubernetes/cloud-config on BOTH master and node

    # patch /etc/systemd/system/kubelet.service.d/10-kubeadm.conf on both VM(s) to add the cloud provider related arguments
    sed -i -E 's/(.*)KUBELET_KUBECONFIG_ARGS=(.*)$/\1KUBELET_KUBECONFIG_ARGS=--cloud-provider=openstack --cloud-            config=\/etc\/kubernetes\/cloud-config \2/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

    systemctl daemon-reload
    systemctl restart kubelet

<br> 3.Run the the script to install on the worker node. Do this for all nodes created.</br>
 
    sudo sh worker.sh

## 3.Work with kubeadm

### Initialize Kubernetes master

We can now use kubeadm to initialize kubernetes on the master Node.

    kubeadm init --config kubeadm.conf

### Install weave for networking

    kubectl --kubeconfig ./admin.conf apply -f https://git.io/weave-kube-1.6
    
### Join worker nodes

Also use kubeadm join to add workers.

    sudo   kubeadm join --token bdc910.dac015f93ad5a064 10.50.0.11:6443
    
If this ran successfully, then you should be able to see the nodes.

    kubectl --kubeconfig ./admin.conf get nodes

### Reset kubeadm

you can reset kubeadm state with:

    kubeadm reset

### Cleaning up

If you need to clean up the Master and the nodes then use this:

    systemctl stop kubelet
    docker rm -f $(docker ps -a -q)
    docker rmi -f $(docker images -q -a)
    kubeadm reset
    systemctl daemon-reload
    systemctl start kubelet

Dims has created some [notes](https://gist.github.com/dims/3914bdfb0ebde52ea0118a237a4d90b9) that I found helpful.
