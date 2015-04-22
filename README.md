# Calico demo using CoreOS Vagrant

This repo is a preconfigured fork of the <a href="https://github.com/coreos/coreos-vagrant">sample CoreOS Vagrant configuration</a> for the <a href="https://github.com/Metaswitch/calico">Project Calico</a> <a href="https://github.com/Metaswitch/calico-docker">docker networking demo</a>.  The modifications are as follows:

* Enable automatic etcd cluster provisioning.
* Set CoreOS version to 'alpha'.
* Set number of nodes to default to 2.
* Preload the Calico docker images on the CoreOS hosts.
* Download the calicoctl executable to the CoreOS hosts.

## Streamlined setup

1) Install dependencies

* [VirtualBox][virtualbox] 4.3.10 or greater.
* [Vagrant][vagrant] 1.6 or greater.
* [Git][git]

2) Clone this project and get it running!

    git clone https://github.com/Metaswitch/calico-coreos-vagrant-example.git
    cd calico-coreos-vagrant-example

3) Startup and SSH

There are two "providers" for Vagrant with slightly different instructions.
Follow one of the following two options:

**VirtualBox Provider**

The VirtualBox provider is the default Vagrant provider. Use this if you are unsure.

    vagrant up

**VMware Provider**

The VMware provider is a commercial addon from Hashicorp that offers better stability and speed.
If you use this provider follow these instructions.

VMware Fusion:

    vagrant up --provider vmware_fusion

VMware Workstation:

    vagrant up --provider vmware_workstation

To connect to your servers
* Linux/Mac OS X
    * run `vagrant ssh <hostname>`
* Windows
    * Follow instructions from https://github.com/nickryand/vagrant-multi-putty
    * run `vagrant putty <hostname>`

4) Verify environment

You should now have two CoreOS servers, each running etcd in a cluster. The servers are named core-01 and core-02.  By default these have IP addresses 172.17.8.101 and 172.17.8.102.

At this point, it's worth checking that your servers can ping each other.

From core-01

    ping 172.17.8.102

From core-02

    ping 172.17.8.101

If you see ping failures, the likely culprit is a problem with the VirtualBox network between the VMs.  You should check that each host is connected to the same virtual network adapter in Virtual Box and rebooting the host may also help.  Remember to shut down the VMs first with `vagrant halt` before you reboot.

You should also verify each host can access etcd.  The following will return an error if etcd is not available.

    etcdctl ls /

5) Get started [using Calico networking][using-calico] and [CoreOS][using-coreos]

[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/downloads.html
[using-coreos]: http://coreos.com/docs/using-coreos/
[using-calico]: https://github.com/Metaswitch/calico-docker/blob/master/docs/GettingStarted.md
[git]: http://git-scm.com/

## Additional optional setup steps

#### Shared Folder Setup

There is optional shared folder setup.
You can try it out by adding a section to your Vagrantfile like this.

    config.vm.network "private_network", ip: "172.17.8.150"
    config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true,  :mount_options   => ['nolock,vers=3,udp']

After a `vagrant reload` you will be prompted for your local machine password.

#### Provisioning with user-data

The Vagrantfile will provision your CoreOS VM(s) with [coreos-cloudinit][coreos-cloudinit] if a `user-data` file is found in the project directory.  coreos-cloudinit simplifies the provisioning process through the use of a script or `cloud-config` document.

Edit `user-data` and make any necessary modifications.
Check out the [coreos-cloudinit documentation][coreos-cloudinit] to learn about the available features.

[coreos-cloudinit]: https://github.com/coreos/coreos-cloudinit

#### Configuration

The Vagrantfile will parse a `config.rb` file containing a set of options used to configure your CoreOS cluster.
See `config.rb` for more information.

## Cluster Setup

Launching a CoreOS cluster on Vagrant is as simple as configuring `$num_instances` in a `config.rb` file to 3 (or more!) and running `vagrant up`.
Make sure you provide a fresh discovery URL in your `user-data` if you wish to bootstrap etcd in your cluster.

## New Box Versions

CoreOS is a rolling release distribution and versions that are out of date will automatically update.
If you want to start from the most up to date version you will need to make sure that you have the latest box file of CoreOS.
Simply remove the old box file and vagrant will download the latest one the next time you `vagrant up`.

    vagrant box remove coreos --provider vmware_fusion
    vagrant box remove coreos --provider vmware_workstation
    vagrant box remove coreos --provider virtualbox

## Docker Forwarding

By setting the `$expose_docker_tcp` configuration value you can forward a local TCP port to docker on each CoreOS machine that you launch. The first machine will be available on the port that you specify and each additional machine will increment the port by 1.

Follow the [Enable Remote API instructions][coreos-enabling-port-forwarding] to get the CoreOS VM setup to work with port forwarding.

[coreos-enabling-port-forwarding]: https://coreos.com/docs/launching-containers/building/customizing-docker/#enable-the-remote-api-on-a-new-socket

Then you can then use the `docker` command from your local shell by setting `DOCKER_HOST`:

    export DOCKER_HOST=tcp://localhost:2375
