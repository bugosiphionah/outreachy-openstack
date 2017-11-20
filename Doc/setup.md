
Setting up

## Running Tests

The first thing I did was upgrade to Go 1.9 since kubernetes I noticed does not work with Go 1.6 because there are imports to context.

I ran first all tests with

	make test

![make test](/images/maketest.png)

For open stack specific tests. I used:

	go test -v k82.io/kubernetes/pkg/cloudprovider/providers/openstack

![make test](/images/openstack.png)

## Run sonobuoy

1.I first ran a kubernetes cluster locally. I was having problems around using the prefered use of minikube where i was failing to get a 32bit installer for ubuntu. I simply downloaded a kubernetes release and ran this command using the instructions [here](https://kubernetes-v1-4.github.io/docs/getting-started-guides/locally/).

	hack/local-up-cluster.sh 

![kubernetes cluster](/images/kube-cluster.png)

2. I then installed **kubectl**

![kubectl](/images/kubectl.png)

3. I then regenerated the quickstart YAML files.

	make generate

![make generate](/images/makegenerate.png)

4. I then tried to deploy a sonobuoy pod to kubernetes cluster with:

	kubectl apply -f examples/quickstart.yaml

I got this error

![sonobuoy error](/images/sonobuoyerr.png)

I solved this by installing kubeadm and running these commands:

	sudo cp /etc/kubernetes/admin.conf $HOME/
	sudo chown $(id -u):$(id -g) $HOME/admin.conf
	export KUBECONFIG=$HOME/admin.conf

I was able to successfully deploy sonobuoy on my cluster and view active pods with:

	kubectl get pods -l component=sonobuoy --namespace=heptio-sonobuoy

![sonobuoy success](/images/sonobuo.png)
