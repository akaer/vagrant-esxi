# Vagrant ESXi Provider

This is a Vagrant plugin for VMware ESXi.

## Goal

It's to continue the stop development of the original plugin and add new features

  This version support new features such as num of cpus, memory size, vmx properties, vagrant box instead use VM cloning.

## Work in progress

Version 0.3.0.

- Use ovftool to import an ova/ovf instead cloning an existing VM
- Add a second hardrive, drive size specified in Mb

Version 0.3.1.

- Add second NIC for public network
- Set IP address for private network (first NIC needed at boot)
- Set IP address for public network (second NIC added by vagrant)

Version 0.3.2

- Replace Open3.capture2 call by Vagrant::Util::Subprocess.execute

Version 0.3.5

- Use disk mode thin during OVA import

Version 0.3.6

- Allow to use an already existing VM as template by cloning
- Allow to create a cloned VM template from the imported OVA to avoid multiple import of the same OVA. Will use the cloned VM instead

## Prerequistes

You must install ovftool on your computer running vagrant.

**NOTE:** This is a work in progress, it's forked from
[vagrant-esxi](https://github.com/swobspace) and originally derived from
[vagrant-vsphere](https://github.com/nsidc/vagrant-vsphere) 
and [vagrant-aws](https://github.com/mitchellh/vagrant-aws).

**WARNING:** vagrant-esxi is for standalone ESXi hosts only. If you are
using a vSphere Server managing datacenters, use 
[vagrant-vsphere](https://github.com/nsidc/vagrant-vsphere). 
Otherwise your datacenter configuration may break.

## Plugin Installation

    git clone https://github.com/Fred78290/vagrant-esxi.git
    cd vagrant-esxi
    gem build vagrant-esxi.gemspec
    vagrant plugin install vagrant-esxi

## ESXi Host Setup

1. enable SSH
2. enable public key authentication, e.g. `cat ~/.ssh/id_rsa.pub | ssh root@host 'cat >> /etc/ssh/keys-root/authorized_keys'`
3. set the license key (if you haven't done so already), e.g. `ssh root@host vim-cmd vimsvc/license --set 'XXXXX-XXXXX-XXXXX-XXXXX-XXXXX'`

## Create a Template VM

This vagrant-esxi uses vagrant box or vagrant box_url to create virtual machines.
You need to install ovftool on your host to allow import vagrant box to your esxi.

You need also create a vagrant box for esxi provider
(i.e. with the vSphere client or VMWare Fusion/Workstation) and make it vagrant compatible. 
For example we create a virtual machine running ubuntu xenial with the name "xenial64".

1. Install at least ssh, sudo, rsync

2. Install vmware-tools provided by your esxi installation. 
You need linux-headers-amd64 (for 64bit architecture), gcc and make. This is essential.
Vagrant will fail if vmware-tools are not up and running.

3. Create the user vagrant: `useradd --comment "Vagrant User" -m vagrant`. You don't need a  password for user `vagrant`.

4. Install the vagrant public key 
from [vagrant insecure keypair](https://github.com/mitchellh/vagrant/tree/master/keys)
as `~vagrant/.ssh/authorized_keys` 
5. Add the following line to /etc/sudoers using the command `visudo`:

   `vagrant ALL=NOPASSWD: ALL`

For additional information see [Vagrant: create a base box](http://docs.vagrantup.com/v2/boxes/base.html) and
[Vagrant: VMware/Boxes](http://docs.vagrantup.com/v2/vmware/boxes.html)

## Example Vagrantfile

    Your box must contains an OVF or OVA file

    Vagrant.configure("2") do |config|
      config.vm.box = "xenial64"
      config.vm.box_url = "http://somewhere.com/xenial64_as_ovf.box"
      config.vm.hostname = "myhost"

      config.vm.provider :esxi do |esxi|
        esxi.network = "VM Network" # The name of the network to attach the created VM
        esxi.name = "XENIAL_VM"
        esxi.host = "host"
        esxi.datastore = "datastore1"
        esxi.user = "root"
        esxi.password = "password"
        esxi.add_hd = 1024
        esxi.memory_mb = 8192
        esxi.cpu_count = 4
        esxi.vmx["sata0.present"] = "FALSE"

        # Features 0.3.6
        esxi.create_template = true # specify to create a clone from the imported OVA, must provide template_name
        esxi.template_name = "AFP-Esxi-Vagrant-Test" # The VM name to use as template
        esxi.clone_template = true # specify to use an existing VM
        esxi.destroy_template = true # specify to destroy the created template VM
      end

      config.vm.network "private_network", ip: "172.16.62.50", nic_type: "e1000", network: "VM Network"
      #config.vm.network "public_network", bridge: "eth1", ip: "10.10.1.5", nic_type: "e1000", network: "VM Public"
      config.vm.network "public_network", ip: "10.10.1.5", nic_type: "e1000", network: "VM Public"
    end

## Issues

https://github.com/Fred78290/vagrant-esxi/issues
