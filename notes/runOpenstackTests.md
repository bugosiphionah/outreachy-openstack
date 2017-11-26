
Setting up

## Running Tests

The first thing I did was upgrade to Go 1.9 since kubernetes I noticed does not work with Go 1.6 because there are imports to context.

I ran first all tests with

	make test

![make test](/images/maketest.png)

For open stack specific tests. I used:

	go test -v k82.io/kubernetes/pkg/cloudprovider/providers/openstack

![make test](/images/openstack.png)


