
# Outreachy Tracker - Go container Related Projects In open stack 

Author: Phionah Bugosi
Email : bugosip@gmail.com 
IRC:	phionah
blog: 	[Tech blog](http://bugosip.blogspot.ug/)

This is a tracker for my outreachy project on Go containers related projects in openstack. I will use this to document some processes now and then.

## Notes

These links will get you to some of the notes I have made as I go about my work for own reference but also to anybody that may find this helpful. I hope I save someone a night debugging openstack and or kubernetes.

1. [Set up openstack development environment](notes/setupopenstack.md).
2. [Execute a suonobuoy run on a kubernetes cluster](notes/runSonobuoy.md)
3. [Run openstack tests in Kubernetes](notes/runOpenstackTests.md)

## Project Details

### Related Repositories

+ [Kubernetes](https://github.com/kubernetes/kubernetes)
+ [Gophercloud](https://github.com/gophercloud/gophercloud)

### BackLog

+ Add Cinder e2e tests.

### Tasks in Progress

+ Add test for Cinder Volume Resize for this [commit](https://github.com/kubernetes/kubernetes/pull/51498/commits/270de26987019ca7442ce1a38e17dbe6a07991f7).

### Finished Tasks

+ Setup openstack environment.
+ Run openstack tests in kubernetes.
+ Setup and Run Sonobuoy.

### Related PRS

+ [Add go vet to Travis](https://github.com/gophercloud/gophercloud/pull/536)
+ [Compute v2: Add Evacuate Action](https://github.com/gophercloud/gophercloud/pull/532)
+ [Compute v2: Add Lock/Unlock Action](https://github.com/gophercloud/gophercloud/pull/522)
+ [SharedFilesystems v2: Change Specs to ExtraSpecs ](https://github.com/gophercloud/gophercloud/pull/517)
+ [Neutron Ports: Add support for extra_dhcp_opt](https://github.com/gophercloud/gophercloud/pull/533)
+ [OpenStack cloud provider should support project instead of tenant](https://github.com/kubernetes/kubernetes/issues/52563)




