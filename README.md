# cRookly

#### Ansible roles & playbooks for deploying simple SUSE CaaS Platform Kubernetes clusters, optionally with SES Rook on top. For anyone interested in playing with local deployments, a sample Vagrantfile is included.

## Table of Contents

* [About](#about-the-project)
* [Getting Started](#getting-started)
	* [Assumptions](#assumptions)
	* [Prerequisites](#prerequisites)
* [Usage](#usage)
	* [Vagrant boxes](#vagrantboxes)
	* [Physicall hosts/Cloud servers](#physcloud)

## About

We provide a couple of Ansible roles and playbooks for quickly deploying simple [SUSE CaaS Platform](https://www.suse.com/products/caas-platform) Kubernetes clusters. After you have your CaaSP cluster up and running, you can optionally install [SES Rook](https://www.suse.com/c/ceph-on-kubernetes-tech-preview) on top. You may try those deployments on bare metal, on cloud servers or on local VMs. For the latter, we include a sample `Vagrantfile` to use with Vagrant.

## Getting Started

You need a Linux or OS X host with [Ansible](https://www.ansible.com) installed. Should you plan on using VMs, then you may use the provided `Vagrantfile` to quickly bring up all necessary cluster nodes. In that case you also need to have [Vagrant](https://www.vagrantup.com) installed on your Linux computer, along with the [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt) plugin. But before we get into any of that, there are some things you ought to have in mind.

### Assumptions

* We deploy CaaSP 5.0, which is based on [SUSE Linux Enterprise Server 15.2](https://www.suse.com/c/suse-linux-enterprise-15-service-pack-2-is-generally-available).
* Should you choose to experiment with local VMs, Vagrant boxes for SLES 15.2 are available from the [SUSE Internal Build Service](http://download.suse.de/ibs/Virtualization:/Vagrant:/SLE-15-SP2/images) (IBS for short).
* All necessary repositories and modules we add to our base Vagrant box/cloud servers/physical hosts, are available from IBS.
* For Rook we use the YAML files from package `rook-k8s-yaml`, coming from the SES 7.0 repositories.
* SES 7.0 repositories are available from IBS also. 
* By now it should be clear, though it doesn't hurt to explicitly mention that access to the Internal Build Service is mandatory.
* Per the official instructions for deploying a CaaSP Kubernetes cluster, we use one virtual/physical machine as a _jump host_. On said host resides [skuba](https://github.com/SUSE/skuba), a CLI tool for managing the full lifecycle of a Kubernetes cluster. More specifically, `skuba` initializes the control plane, bootstraps the Master nodes, and allows the Worker nodes to join the cluster.
* Our whole Kubernetes setup is intentionally simple, meaning we do not have any load balancers and we prepare one Master node only. In addition to the jump host and the Master node, we have three Worker nodes joining the cluster. Of course, you can easily have two or more than three Worker nodes joining. Theoretically you can even have only one Worker node joining the cluster, but it should be noted that at the time of writing we haven't tested such a configuration.

### Prerequisites

First off, make sure you have [Ansible](https://www.ansible.com) installed. If needed, check the [installation instructions](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) for your host OS.

If you plan on testing/playing/experimenting locally, you need to be on a Linux host and have [Vagrant](https://www.vagrantup.com) installed, along with the [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt) plugin. Additionally, you need KVM/QEMU and libvirt, and your user has to be a member of the `libvirt` group. In case you're running openSUSE Leap 15.1/15.2, here are the steps for setting everything up:

```
> sudo zypper -n in -t pattern kvm_server kvm_tools
> sudo systemctl enable libvirtd
> sudo systemctl restart libvirtd
> sudo usermod -a -G libvirt $USER
```

Log out -- then log back in. Your user should now be a member of the `libvirt` group (verify by looking at the output of the `groups` command). For Vagrant and the vagrant-libvirt plugin, execute the following:

```
# in the repository URL substitute <ver> for 15.1 or 15.2,
# depending on the version of openSUSE Leap you're running

> sudo zypper ar \
	https://download.opensuse.org/repositories/Virtualization:/vagrant/openSUSE_Leap_<ver> \
	vagrant_repo
> sudo zypper ref
> sudo zypper -n in vagrant vagrant-libvirt
```

Verify that all required software and plugins are readily available:

```
> ansible --version

ansible 2.9.11
  config file = /home/sub0/colder/ansible-playground/ansible.cfg
  configured module search path = ['/home/sub0/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.6/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.6.10 (default, Jan 16 2020, 09:12:04) [GCC]
```

```
> vagrant --version

Vagrant 2.2.9
```

```
> vagrant plugin list

vagrant-libvirt (0.0.45, system)
```

## Usage

TODO

### Vagrant boxes

TODO

### Physicall hosts/Cloud servers

TODO