# How to use kubeadm with openstack

Kubernetes is an open-source platform designed to automate deploying, scaling, and operating application containers.
It eliminates many of the manual processes involved in deploying and scaling containerized applications.

In order for us to use kubernetes, we have to install and run it some where like an operating system or cloud. The default installer
for kubernetes is kubeadm. 

In this post, I will describe deployment of kubernets 1.9 to openstack using kubeadm. I will cover

+ Creating network instances on openstack to host kubernetes
+ Install kubernetes and docker
+ Install master Kubernetes \n                                                                                                      sudo kubeadm init
+ Join worker nodes                                                                                                      
    sudo   kubeadm join
    



