
# SetUp openstack development environment

As documented by openstack, the easiest way to setup a locally is by using devstack. On the openstack documentation we are told that the steps there have been most tested on ubuntu 16.04. I would recommend using ubuntu 16.04 to simply your life.

These notes detail a procedure I used to setup an openstack development environment on ubuntu 16.04.

## A hint the open stack documentation  dint warn you about OS (ubuntu)

Install a **64 bit version** of ubuntu because as of writing this, **./stash.sh** will not work on a 32 bit system of ubuntu 16.04.

Let me get to the magic that worked for me.

## Steps to install openstack dev environment

1. Add stack user

	sudo useradd -s /bin/bash -d /opt/stack -m stack

2. Give the user sudo previledges to make system changes.
	
	echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
	sudo su - stack

3. Get devstack
	
	git clone https://git.openstack.org/openstack-dev/devstack
	cd devstack

4. create local.conf file to store passwords at the rootof devstack

	sudo touch local.conf

5. Edit local.conf file adding 4 passwords

	[[local|localrc]]
	ADMIN_PASSWORD=secret
	DATABASE_PASSWORD=$ADMIN_PASSWORD
	RABBIT_PASSWORD=$ADMIN_PASSWORD
	SERVICE_PASSWORD=$ADMIN_PASSWORD

6. Do Installation

	./stack.sh

![make test](/images/openstackenv.png)

## Common hickups you might face

### [ERROR] /opt/devstack/functions-common:1077 Failed to update apt repos, we're dead now

This is a common error what you need to do is first ensure sudo update is working flawlessly. use the  procedure below to get it working well.

	apt-get clean
	rm -rf /var/lib/apt/lists/*
	apt-get clean
	apt-get update 
	apt-get upgrade

I also had to edit the file /opt/devstack/functions-common.

1. open the file with your fav editor

	subl functions-common

2. Look for this line of code in that file.

	if ! timeout 300 sh -c "while ! $update_cmd; do sleep 30; done"; then

Then change 300 to 1000


## Conclusion

This is what worked for me. If you have a different story or you want some help from me to debug this, then ping me on bugosip@gmail.com.