## Run sonobuoy

1. Install **kubectl**

	curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

![kubectl](/images/kubectldownload.png)
![kubectl](/images/kubectldownload.png)

2. Run a kubernetes cluster locally with minukube.

This is the recommended way.

First you will need to [install minikube]().

	curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.23.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

Run your kubernetes cluster

	minikube start 

![kubernetes cluster](/images/minikubestart.png)


3. I regenerate the quickstart YAML files.

	make generate

![make generate](/images/makegenerate.png)

4. I deployed a sonobuoy pod to kubernetes cluster with:

	kubectl apply -f examples/quickstart.yaml


![sonobuoy success](/images/quickstartyml.png)

5. You can then list available pods.

	kubectl get pods -l component=sonobuoy --namespace=heptio-sonobuoy

![sonobuoy list](/images/listpods.png)

## Common Errors

I got the error below while trying step 4.

![sonobuoy error](/images/sonobuoyerr.png)

I solved this by installing kubeadm and running these commands:

	sudo cp /etc/kubernetes/admin.conf $HOME/
	sudo chown $(id -u):$(id -g) $HOME/admin.conf
	export KUBECONFIG=$HOME/admin.conf

I was able to successfully deploy sonobuoy on my cluster after this.