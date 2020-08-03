# cRookly

#### Ansible roles & playbooks for deploying simple SUSE CaaS Platform Kubernetes clusters, optionally with SES Rook on top. For anyone interested in playing with local deployments, a sample Vagrantfile is also included.

## Table of Contents

* [About](#about-the-project)
* [Getting Started](#getting-started)
	* [Assumptions](#assumptions)
	* [Prerequisites](#prerequisites)
* [Usage](#usage)
	* [Vagrant boxes](#vagrantboxes)
	* [Physical hosts/Cloud servers](#physcloud)
	* [Control from localhost](#ctrllocalhost)

## About

We provide a couple of Ansible roles and playbooks for quickly deploying simple [SUSE CaaS Platform](https://www.suse.com/products/caas-platform) Kubernetes clusters. After you have your CaaSP cluster up and running, you may optionally install [SES Rook](https://www.suse.com/c/ceph-on-kubernetes-tech-preview) on top. You can try those deployments on bare metal, on cloud servers or on local VMs. For the latter, we include a sample `Vagrantfile` to use with Vagrant.

## Getting Started

You need a Linux or Mac OS X host with Ansible installed. Should you plan on using VMs and you're on a Linux host, one of your options is to use Vagrant with the provided `Vagrantfile`. But before we get into any of that, there are some things you have to have in mind.

### Assumptions

* We deploy CaaSP 5.0, which is based on [SUSE Linux Enterprise Server 15.2](https://www.suse.com/c/suse-linux-enterprise-15-service-pack-2-is-generally-available).
* Should you choose to experiment with local VMs, Vagrant boxes for SLES 15.2 are available from the [SUSE Internal Build Service](http://download.suse.de/ibs/Virtualization:/Vagrant:/SLE-15-SP2/images) (IBS).
* All necessary repositories and modules we add to our base Vagrant box/cloud servers/physical hosts, are available from IBS.
* For Rook we use the YAML files from package `rook-k8s-yaml`, coming from the SES 7.0 repositories on IBS.
* Per the official instructions for deploying a CaaSP cluster, we employ one virtual/physical machine as a _jump host_. On that host resides [skuba](https://github.com/SUSE/skuba), a CLI tool for managing the full lifecycle of a Kubernetes cluster. More specifically, `skuba` initializes the control plane, bootstraps the Master nodes, and allows the Worker nodes to join the cluster.
* Our whole Kubernetes setup is intentionally simple, meaning we do not have any load balancers and we prepare one Master node only. In addition to the jump host and the Master node, we have three Worker nodes joining the cluster. Of course, you can easily have two or more than three Worker nodes joining. Theoretically you can even have only one Worker node joining the cluster, but it should be noted that at the time of writing we haven't tested such a configuration.

### Prerequisites

First off, make sure you have [Ansible](https://www.ansible.com) installed. If needed, check the [installation instructions](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) for your host OS.

If you plan on testing/playing/experimenting locally, you need to be on a Linux host and have [Vagrant](https://www.vagrantup.com) installed, along with the [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt) plugin. Additionally, you need KVM/QEMU and libvirt, and your user has to be a member of the `libvirt` group. In case you're running openSUSE Leap 15.1/15.2, here are the steps for setting everything up:

```
~> sudo zypper -n in -t pattern kvm_server kvm_tools
~> sudo systemctl enable libvirtd
~> sudo systemctl restart libvirtd
~> sudo usermod -a -G libvirt $USER
```

Log out -- then log back in. Your user should now be a member of the `libvirt` group (verify by looking at the output of the `groups` command). For Vagrant and the vagrant-libvirt plugin, execute the following:

```
~> sudo zypper ar \
	https://download.opensuse.org/repositories/Virtualization:/vagrant/openSUSE_Leap_<ver> \
	vagrant_repo

# in the repository URL substitute <ver> for 15.1 or 15.2,
# depending on the version of openSUSE Leap you're running

~> sudo zypper ref
~> sudo zypper -n in vagrant vagrant-libvirt
```

Verify that all required software and plugins are readily available:

```
~> ansible --version

ansible 2.9.11
  config file = /home/sub0/colder/ansible-playground/ansible.cfg
  configured module search path = ['/home/sub0/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.6/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.6.10 (default, Jan 16 2020, 09:12:04) [GCC]
```

```
~> vagrant --version

Vagrant 2.2.9
```

```
~> vagrant plugin list

vagrant-libvirt (0.0.45, system)
```

## Usage

Have the jump host, the Master node and all the Worker nodes up and running. Make sure you have key-based SSH access to the same user account on each and every single one of them. If that account is not the root account, then it should have passwordless sudo rights.

On the host you're going to be working on, clone the cRookly repository and change into it. The Ansible hosts file should accurately reflect your jump host and the nodes, so take a look at `ansible/inventory/caasp` and make any changes deemed necessary.

```
[caasp_jumphost]
jhost   ansible_host=192.168.10.99

[caasp_master]
master  ansible_host=192.168.10.100

[caasp_workers]
worker1 ansible_host=192.168.10.101
worker2 ansible_host=192.168.10.102
worker3 ansible_host=192.168.10.103

[caasp_cluster:children]
caasp_master
caasp_workers

[all:vars]
ansible_ssh_user=vagrant
ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
```

Please do not modify the names of the Ansible groups (`caasp_jumphost`, `caasp_master`, `caasp_workers` and `caasp_cluster`), nor the Ansible hostnames of the jump host and Master node (`jhost` and `master` respectively).

Delete or add any Worker nodes if necessary, preferably following the naming convention in the inventory.

Take care of the IP addresses, especially if you're not going to deploy on Vagrant boxes. Even then, other VMs may be in subnet `192.168.10.0/24`, so decide on a different subnet and modify the inventory file accordingly.

If you're not going to use Vagrant, in the `all` section of the inventory make sure you set `ansible_ssh_user` to the username of a certain remote user. That would be the user with passwordless sudo rights, in their account you have key-based SSH access to. Your local user's public SSH key will be in the `~/.ssh/authorized_keys` of any of those remote accounts, hence variable `ansible_ssh_private_key_file` should be set to the full path of the corresponding local private key.

### Vagrant boxes

Change into the `vagraland` directory and take a look at the beginning of the `Vagrantfile`:

```
[...]

the_box = 'sles-15-sp2'
libvirt_pool = 'default'

cluster_net_prefix = '192.168.10'
first_node_addr = 99

worker_count = 3
osd_disk_capacity = '32G'
osd_disks_per_worker = 3

ram_size = 4096
core_count = 4

[...]
```

Variable `the_box` holds the name of a local Vagrant box for libvirt, with SLES 15 SP2. Such boxes we can add to our host from IBS:

```
~/crookly/vagraland> vagrant box add --name sles-15-sp2 --provider libvirt \
	http://download.suse.de/ibs/Virtualization:/Vagrant:/SLE-15-SP2/images/SLES15-SP2-Vagrant.x86_64-libvirt.box
```
```
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'sles-15-sp2' (v0) for provider: libvirt
    box: Downloading: http://download.suse.de/ibs/Virtualization:/Vagrant:/SLE-15-SP2/images/SLES15-SP2-Vagrant.x86_64-libvirt.box
==> box: Successfully added box 'sles-15-sp2' (v0) for 'libvirt'!
```
```
~/crookly/vagraland> vagrant box list
```
```
sles-15-sp2 (libvirt, 0)
```

Depending on the resources of your physical host and the number of Worker nodes you're about to create (`worker_count`), you might decide to decrease `ram_size` (expressed in megabytes) and/or `core_count`.

Now, to bring up all nodes, just type the following:

```
~/crookly/vagraland> vagrant up
```
```
Bringing machine 'jhost' up with 'libvirt' provider...
Bringing machine 'master' up with 'libvirt' provider...
Bringing machine 'worker1' up with 'libvirt' provider...
Bringing machine 'worker2' up with 'libvirt' provider...
Bringing machine 'worker3' up with 'libvirt' provider...

[...]
```

Soon your VMs will be up and running and ready to be provisioned:

```
~/crookly/vagraland> vagrant status
```
```
Current machine states:

jhost                     running (libvirt)
master                    running (libvirt)
worker1                   running (libvirt)
worker2                   running (libvirt)
worker3                   running (libvirt)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

Switch to the `ansible` directory and deploy CaaSP with just a one-liner:

```
~/crookly/ansible> ansible-playbook -i inventory/caasp caasp.yml
```

Our Vagrant box from IBS has no repositories so during deployment we add to each node the ones we need -- specifically all repositories specified in `vars/caasp.yml`:

```
domain_name: 'caasp.is'

repos:
  - { name: 'prod', url: 'http://download.suse.de/ibs/SUSE/Products/SLE-Product-SLES/15-SP2/x86_64/product' }
  - { name: 'prod_up', url: 'http://download.suse.de/ibs/SUSE/Updates/SLE-Product-SLES/15-SP2/x86_64/update' }

  - { name: 'base_sys', url: 'http://download.suse.de/ibs/SUSE/Products/SLE-Module-Basesystem/15-SP2/x86_64/product' }
  - { name: 'base_sys_up', url: 'http://download.suse.de/ibs/SUSE/Updates/SLE-Module-Basesystem/15-SP2/x86_64/update' }

  - { name: 'srv_app', url: 'http://download.suse.de/ibs/SUSE/Products/SLE-Module-Server-Applications/15-SP2/x86_64/product' }
  - { name: 'srv_app_up', url: 'http://download.suse.de/ibs/SUSE/Updates/SLE-Module-Server-Applications/15-SP2/x86_64/update' }

  - { name: 'contain', url: 'http://download.suse.de/ibs/SUSE/Products/SLE-Module-Containers/15-SP2/x86_64/product' }
  - { name: 'contain_up', url: 'http://download.suse.de/ibs/SUSE/Updates/SLE-Module-Containers/15-SP2/x86_64/update' }

  - { name: 'caasp', url: 'http://download.suse.de/ibs/SUSE/Products/SUSE-CAASP/5/x86_64/product' }
  - { name: 'caasp_up', url: 'http://download.suse.de/ibs/SUSE/Updates/SUSE-CAASP/5/x86_64/update' }
```
After Ansible has finished provisioning the nodes, we can SSH into the root account in our jump host (`ssh root@192.168.10.99` -- password is `vagrant`) and take a look of our new CaaSP cluster using `kubectl`:

```
jhost:~ # kubectl get nodes
```
```
NAME      STATUS   ROLES    AGE     VERSION
master    Ready    master   7m58s   v1.18.6
worker1   Ready    <none>   5m39s   v1.18.6
worker2   Ready    <none>   3m38s   v1.18.6
worker3   Ready    <none>   77s     v1.18.6
```

```
jhost:~ # kubectl get pods --all-namespaces -o wide
```
```
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE     IP               NODE      NOMINATED NODE   READINESS GATES
kube-system   cilium-4n9t8                       1/1     Running   0          4m36s   192.168.10.102   worker2   <none>           <none>
kube-system   cilium-7j955                       1/1     Running   0          8m6s    192.168.10.100   master    <none>           <none>
kube-system   cilium-dqm5z                       1/1     Running   0          6m37s   192.168.10.101   worker1   <none>           <none>
kube-system   cilium-operator-567fb5c9bf-5xlth   1/1     Running   0          8m7s    192.168.10.101   worker1   <none>           <none>
kube-system   cilium-x9tvv                       1/1     Running   0          2m16s   192.168.10.103   worker3   <none>           <none>
kube-system   coredns-84d6b58b57-hbks2           1/1     Running   0          8m10s   10.244.0.134     master    <none>           <none>
kube-system   coredns-84d6b58b57-vzfwl           1/1     Running   0          8m10s   10.244.0.18      master    <none>           <none>
kube-system   etcd-master                        1/1     Running   0          7m29s   192.168.10.100   master    <none>           <none>
kube-system   kube-apiserver-master              1/1     Running   0          7m30s   192.168.10.100   master    <none>           <none>
kube-system   kube-controller-manager-master     1/1     Running   3          7m40s   192.168.10.100   master    <none>           <none>
kube-system   kube-proxy-8kjn7                   1/1     Running   0          8m11s   192.168.10.100   master    <none>           <none>
kube-system   kube-proxy-9q9c4                   1/1     Running   0          4m36s   192.168.10.102   worker2   <none>           <none>
kube-system   kube-proxy-lxhfq                   1/1     Running   0          2m16s   192.168.10.103   worker3   <none>           <none>
kube-system   kube-proxy-npwzg                   1/1     Running   0          6m37s   192.168.10.101   worker1   <none>           <none>
kube-system   kube-scheduler-master              1/1     Running   3          7m41s   192.168.10.100   master    <none>           <none>
kube-system   kucero-sjn6s                       1/1     Running   0          7m5s    10.244.0.173     master    <none>           <none>
kube-system   kured-8s4ql                        1/1     Running   0          2m35s   10.244.2.43      worker2   <none>           <none>
kube-system   kured-jtblb                        1/1     Running   0          7m5s    10.244.0.105     master    <none>           <none>
kube-system   kured-rv5kv                        0/1     Running   0          90s     10.244.3.8       worker3   <none>           <none>
kube-system   kured-tggjs                        1/1     Running   0          4m58s   10.244.1.221     worker1   <none>           <none>
kube-system   metrics-server-8575459f57-mg4gl    1/1     Running   0          6m25s   10.244.0.2       master    <none>           <none>
kube-system   metrics-server-8575459f57-wwstp    1/1     Running   0          6m33s   10.244.1.146     worker1   <none>           <none>
kube-system   oidc-dex-5f848b64f4-5922v          1/1     Running   0          2m24s   10.244.1.196     worker1   <none>           <none>
kube-system   oidc-dex-5f848b64f4-gnt8m          1/1     Running   0          4m30s   10.244.0.209     master    <none>           <none>
kube-system   oidc-dex-5f848b64f4-vdmqd          1/1     Running   0          4m31s   10.244.2.193     worker2   <none>           <none>
kube-system   oidc-gangway-7b5f996b69-dfrv5      1/1     Running   0          4m22s   10.244.0.231     master    <none>           <none>
kube-system   oidc-gangway-7b5f996b69-mw5dq      1/1     Running   0          4m30s   10.244.1.161     worker1   <none>           <none>
kube-system   oidc-gangway-7b5f996b69-tv7mq      1/1     Running   0          4m2s    10.244.2.191     worker2   <none>           <none>
```

To install SES Rook, from our physical host and the `ansible` directory, we type:

```
~/crookly/ansible> ansible-playbook -i inventory/caasp rook.yml
```

This time we add two new repositories on the jump host only -- specifically the ones defined in `vars/ses.yml`:

```
repos:
  - { name: 'ses', url: 'http://download.suse.de/ibs/SUSE/Products/Storage/7/x86_64/product' }
  - { name: 'ses_up', url: 'http://download.suse.de/ibs/SUSE/Updates/Storage/7/x86_64/update' }
```

Again, we can SSH into the jump host and check the state of Rook like this:

```
jhost:~ # kubectl get pods -n rook-ceph -o wide
```
```
NAME                                                READY   STATUS      RESTARTS   AGE   IP               NODE      NOMINATED NODE   READINESS GATES
csi-te-g2v7l                              3/3     Running     0          54m   192.168.10.102   worker2   <none>           <none>
csi-cephfsplugin-provisioner-67c7548794-dhvk9       5/5     Running     0          54m   10.244.1.138     worker1   <none>           <none>
csi-cephfsplugin-provisioner-67c7548794-twjrw       5/5     Running     0          54m   10.244.3.3       worker3   <none>           <none>
csi-cephfsplugin-x9cn8                              3/3     Running     0          54m   192.168.10.103   worker3   <none>           <none>
csi-cephfsplugin-xlm8g                              3/3     Running     0          54m   192.168.10.101   worker1   <none>           <none>
csi-rbdplugin-jlvs8                                 3/3     Running     0          54m   192.168.10.101   worker1   <none>           <none>
csi-rbdplugin-provisioner-5d45fc57b8-8dg5f          6/6     Running     0          54m   10.244.2.138     worker2   <none>           <none>
csi-rbdplugin-provisioner-5d45fc57b8-bg9mc          6/6     Running     0          54m   10.244.3.109     worker3   <none>           <none>
csi-rbdplugin-rrghs                                 3/3     Running     0          54m   192.168.10.102   worker2   <none>           <none>
csi-rbdplugin-xpxn6                                 3/3     Running     0          54m   192.168.10.103   worker3   <none>           <none>
rook-ceph-crashcollector-worker1-56bc7f6788-g97n6   1/1     Running     0          52m   10.244.1.117     worker1   <none>           <none>
rook-ceph-crashcollector-worker2-fd67f6989-pkv54    1/1     Running     0          52m   10.244.2.180     worker2   <none>           <none>
rook-ceph-crashcollector-worker3-57f59cc678-8rgt2   1/1     Running     0          48m   10.244.3.85      worker3   <none>           <none>
rook-ceph-mgr-a-bc6f698-b2pz7                       1/1     Running     0          51m   10.244.3.33      worker3   <none>           <none>
rook-ceph-mon-a-7d68f5c49f-jfsdk                    1/1     Running     0          52m   10.244.2.3       worker2   <none>           <none>
rook-ceph-mon-b-6c9889df9b-rk2qs                    1/1     Running     0          52m   10.244.1.10      worker1   <none>           <none>
rook-ceph-mon-c-5dddbfdd94-8v7j6                    1/1     Running     0          51m   10.244.3.90      worker3   <none>           <none>
rook-ceph-operator-69b98d7547-qxd7j                 1/1     Running     0          57m   10.244.3.2       worker3   <none>           <none>
rook-ceph-osd-0-57b99d75f6-7nctx                    1/1     Running     0          49m   10.244.1.43      worker1   <none>           <none>
rook-ceph-osd-1-6d6bdb8d47-dmt2b                    1/1     Running     0          49m   10.244.2.124     worker2   <none>           <none>
rook-ceph-osd-2-565f7b556-zqxq8                     1/1     Running     0          48m   10.244.3.234     worker3   <none>           <none>
rook-ceph-osd-3-6599ff786b-w4mnr                    1/1     Running     0          49m   10.244.1.115     worker1   <none>           <none>
rook-ceph-osd-4-6cfdf6c499-hth24                    1/1     Running     0          49m   10.244.2.30      worker2   <none>           <none>
rook-ceph-osd-5-69b946c4d8-gmz8p                    1/1     Running     0          48m   10.244.3.215     worker3   <none>           <none>
rook-ceph-osd-6-558d7f9f75-lxwql                    1/1     Running     0          49m   10.244.2.177     worker2   <none>           <none>
rook-ceph-osd-7-584fb94758-7ccqr                    1/1     Running     0          49m   10.244.1.209     worker1   <none>           <none>
rook-ceph-osd-8-7cf6897dcb-7wd74                    1/1     Running     0          48m   10.244.3.49      worker3   <none>           <none>
rook-ceph-osd-prepare-worker1-lfcfs                 0/1     Completed   0          51m   10.244.1.14      worker1   <none>           <none>
rook-ceph-osd-prepare-worker2-fsx9k                 0/1     Completed   0          51m   10.244.2.157     worker2   <none>           <none>
rook-ceph-osd-prepare-worker3-9z8pn                 0/1     Completed   0          50m   10.244.3.7       worker3   <none>           <none>
rook-ceph-tools-764b5dbc7d-bbqlk                    1/1     Running     0          54m   10.244.3.12      worker3   <none>           <none>
rook-discover-f2svf                                 1/1     Running     0          57m   10.244.2.186     worker2   <none>           <none>
rook-discover-n7j5g                                 1/1     Running     0          57m   10.244.3.81      worker3   <none>           <none>
rook-discover-q4ncm                                 1/1     Running     0          57m   10.244.1.114     worker1   <none>           <none>
```

### Physical hosts/Cloud servers

Provided the Ansible hosts file in `ansible/inventory/caasp` accurately reflects your physical hosts/cloud servers (see [Usage](#usage)), you can simply change into the `ansible` directory and deploy CaaSP like this:

```
~/crookly/ansible> ansible-playbook -i inventory/caasp caasp.yml
```

If for some reason you do not need to add repositories, issue the following command instead:

```
~/crookly/ansible> ansible-playbook -i inventory/caasp --skip-tags add_repos caasp.yml
```

Finally, to install Rook just type...

```
~/crookly/ansible> ansible-playbook -i inventory/caasp rook.yml
```

...or just...

```
~/crookly/ansible> ansible-playbook -i inventory/caasp --skip-tags add_repos rook.yml
```

...depending on whether you wish to add SES repositories on the jump host or not.

### Control from localhost

Optionally, you may install a recent version of `kubctl` on your localhost and control the newly created CaaSP cluster and/or Rook installation from the comfort of your host computer. You just need to have `/root/.kube/config` from the the jump host:

```
~> mkdir ~/.kube
~> scp root@192.168.10.99:~/.kube/config ~/.kube/
```
```
~> kubectl get nodes
```
```
NAME      STATUS   ROLES    AGE    VERSION
master    Ready    master   116m   v1.18.6
worker1   Ready    <none>   114m   v1.18.6
worker2   Ready    <none>   112m   v1.18.6
worker3   Ready    <none>   110m   v1.18.6
```

You can issue "standard" commands to the Rook/Ceph cluster, via the Toolbox pod. First, you need the exact name of said pod:

```
~> toolbox_name=$(kubectl -n rook-ceph get pod \
		-l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}')
```

Then, you issue commands like this:

```
~> kubectl -n rook-ceph exec "$toolbox_name" -- ceph status
```
```
  cluster:
    id:     7809f6f9-23e4-4736-b2b0-39d150b85e29
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum a,b,c (age 104m)
    mgr: a(active, since 46m)
    osd: 9 osds: 9 up (since 101m), 9 in (since 101m)
 
  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   9.0 GiB used, 279 GiB / 288 GiB avail
    pgs:     1 active+clean
```
