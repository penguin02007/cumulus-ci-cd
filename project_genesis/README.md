# Cumulus CI/CD for Project Genesis

This repository consists instruction to build a Cumulus Lab on testing with NetQ.

---

## Table of Conents
* [Prerequisite](#rerequisite)
* [Install NetQ](#install-netq)
  * [Setup](#setup)
    * [Networking](#networking)
* [References](#references)

## Prerequisite
This Lab has been tested with the following Hardware and OS:
Intel(R) Xeon(R) CPU E5-2603 v4 @ 1.70GHz
64GB
Ubuntu 18.04
Vagrant 2.2.7
```
apt-get install -y curl qemu-kvm libvirt-bin virtualbox qemu-kvm libvirt-bin libvirt bridge-utils
cd /tmp && curl -O https://releases.hashicorp.com/vagrant/2.2.7/vagrant_2.2.7_x86_64.deb
./vagrant_2.2.7_x86_64.deb

```
## Install NetQ
```
virt-install --name=netq1 --vcpus=8 --memory=32768 --os-type=linux \
--os-variant=debian7 \
--disk path=/opt/apps/downloads/netq-2.4.0-ubuntu-18.04-ts-qemu.qcow2,format=qcow2,bus=virtio,cache=none \
--network=type=direct,source=eno1,model=virtio \
--import \
--noautoconsole
```

### Setup
#### Networking
For NetQ to communicate to the oob-mgmt-switch. A UDP unicast tunnel is required, this provides a virtual network which enables connections between QEMU instances using QEMU's UDP infrastructure.

The xml "source" address is the endpoint address to which the UDP socket packets will be sent from the host running QEMU. The xml "local" address is the address of the interface from which the UDP socket packets will originate from the QEMU host. 
```
...
   <interface type='udp'>
      <mac address='44:38:39:00:01:00'/>
      <source address='127.0.0.1' port='33701'>
        <local address='127.0.0.1' port='33951'/>
      </source>
      <target dev='eth1'/>
      <model type='virtio'/>
      <alias name='net1'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x02' function='0x0'/>
    </interface>
...
```

## References
1. https://libvirt.org/formatdomain.html#elementsNICSUDP