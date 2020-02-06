# Cumulus CI/CD for Project Genesis

This repository consists instruction to build a Cumulus Lab on testing with NetQ.

---

## Table of Conents
* [Prerequisite](#rerequisite)
  * [NetQ Install](#netq-install)

## NetQ Install
```
virt-install --name=netq1 --vcpus=8 --memory=32768 --os-type=linux \
--os-variant=debian7 \
--disk path=/opt/apps/downloads/netq-2.4.0-ubuntu-18.04-ts-qemu.qcow2,format=qcow2,bus=virtio,cache=none \
--network=type=direct,source=eno1,model=virtio \
--import \
--noautoconsole
```

## Connect NetQ to oob-mgmt-switch
To connect NetQ to the oob-mgmt-switch. A UDP unicast tunnel is required, this provides a virtual network which enables connections between QEMU instances using QEMU's UDP infrastructure.

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