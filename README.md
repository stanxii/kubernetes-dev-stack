# Kubernetes Dev-Stack

## Background
Small proof of concept for running kubernetes cluster, specifically intended for development environment. Can create kubernetes cluster compromised of one master and arbitrary number of minions. Can run on Linux, Windows or Mac. Vagrant box is based on Centos 7.1 with latest stable kernel 4.3.0, docker 1.8.2 using overlay storage driver, backed by xfs file system. SELinux will be set to permissive mode, and firewall will be down.

**This is little demo was created for the purpose of development, and should not be used in production, as kubernetes is configured without security.**

### Requirements
1. VirtualBox, recent version, you can get it here: [Oracle Virtual box](http://www.vagrantup.com)
2. Vagrant, recent version, you can get it from here: [Vagrant](http://www.vagrantup.com)

### Getting started
In order to get started, first we need to start kube master, after which we can start multiple minions anywhere on the network as long as they can reach master.

#### Starting kube master
To start kubernetes master (which will also be used to schedule docker containers), do:


    cd vagrant/kube-master
    ## By default vagrant box will use 4G of RAM,
    ## to use less or more, export env variable MEM_SIZE,
    ## example: export MEM_SIZE=8096 for 8Gig VM

    ## Start the VM
    vagrant up --provider=virtualbox

After initial download of vagrant box (once off download, 450Mb) from vagrant repository, box will be automatically configured, and depending on network setup on your machine, it might ask you which network interface you wish to use - normally choose one you use to connect to Internet (normally choice #1 is what you need), but can vary depending on the machine.

Since kubernetes operates on separate network, script to create route to your newly created kubernetes cloud will be generated in the same dir, so run:

    ## Depending on your os, for example Linux:
    ./add-route-LIN.sh
    ## Which will create route
    using your VM as gateway.


You can now ssh into your kubernetes master with:

      vagrant ssh

Kubernetes master should be up and running for you:

    kubectl get nodes
    kubectl get po --namespace=kube-system
    kubectl cluster-info
    ## You can start kube-ui or grafana as example:
    sudo kubectl create -f /etc/kubernetes/kube-ui/
    ## Or Graphana:
    sudo kubectl create -f /etc/kubernetes/grafana/

    ## And monitor progress with:
    watch -n 5 kubectl get po --namespace=kube-system

    ## Once up and running cluster-info will tell you where to go:
    ## So do:
    kubectl cluster-info
    ## and open up Grafana url shown

will confirm it, also there will be dns up and running - depending how fast your network is, it might take a few for docker to pull required images.  

#### Starting kube minion(s)

Change directory to kube-minion:

    cd kubernetes-dev-stack/vagrant/kube-minion
    ## Set the MASTER_IP to point to your kubeernetes master

    export MASTER_IP= ip address of your master

    ## Set MEM_SIZE if you wish more or less then 4Gig for ## minion(s)
    ## Set NUM_MINIONS=n, where n is number of minions you wish to ## start

Vagrant will start up your minions and salt-stack will configure them correctly. Again, depending on your network setup, you might be asked to select network interface over which minions will communincate (normally one you use to access Internet, normally choice #1).

#### Master and minions on separate machines

Since master and minions will be bridged to your host interface they can be on different hosts, only thing required is for the minions to export MASTER_IP as shown above.

#### How it works

Packer template provided in the repo is used to create vagrant box, in case you wish to create your own. Code here will use one I have already created and deployed to vagrant repository.

Salt-stack is used to configure VM upon startup, you can find configuration in salt directory.

Happy hacking.
Dejan