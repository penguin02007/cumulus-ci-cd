# Cumulus CI/CD for Project Genesis

This repository consists instruction to build a Cumulus Lab on testing with NetQ.

---

## Table of Conents
* [Requirements](#requirements)
* [Install NetQ](#install-netq)
  * [Verify hardware](#verify-hardware)
  * [Networking](#networking)
    * [Configure UDP Tunnel](#configure-udp-tunnel)
      * [netq](#netq)
* [References](#references)

## Requirements
The following are a list of recommended software, specs and hardware to run this project successfully.
* **Ubuntu 18.04**
* **64GB Memory**
* **[Vagrant](https://releases.hashicorp.com/vagrant)** - This has been tested to work on 2.2.7.
* **[Ansible](http://ansible.com)**
```
apt-get install -y curl qemu-kvm libvirt-bin virtualbox qemu-kvm bridge-utils virtinst
cd /tmp && curl -O https://releases.hashicorp.com/vagrant/2.2.7/vagrant_2.2.7_x86_64.deb
apt install ./vagrant_2.2.7_x86_64.deb

```

## Install NetQ
```
virt-install --name=netq1 --vcpus=8 --memory=63228 --os-type=linux \
--os-variant=debian7 \
--disk path=/opt/apps/downloads/netq-2.4.0-ubuntu-18.04-ts-qemu.qcow2,format=qcow2,bus=virtio,cache=none \
--network=type=direct,source=eno1,model=virtio \
--import \
--noautoconsole
```

### Verify hardware
```
oot@netq1:~# opta-check

checking connectivity to eth0...
INFO: IP address 172.20.232.68 is reachable
PASS

checking connectivity to eth1...
INFO: IP address 192.168.200.13 is reachable
PASS

Checking for IP address change since last orchestrated in /home/cumulus/.kube/config
INFO: 172.20.232.68 matches IP address of eth0
Checking for IP address change since last orchestrated in /etc/kubernetes/admin.conf
INFO: 172.20.232.68 matches IP address of eth0
PASS

checking default route...
PASS

checking RAM requirements...
INFO: detected 62 GB RAM
PASS

checking CPU requirements...
INFO: detected 8 CPU cores
PASS

PASS
-------
SUMMARY
INFO: running on a SSD is recommended
INFO: ALL CHECKS PASSED
```


### Networking
#### Configure UDP Tunnel
##### NetQ
For NetQ to communicate to oob-mgmt-switch. A UDP unicast tunnel is required to provide a virtual network which enables connections between QEMU instances using QEMU's UDP infrastructure. (Similar to Point-to-Point)

```
virsh dumpxml netq1 > /tmp/netq1.xml
virsh edit netq1
```
```
# netq1 - replace source address and local address
...
   <interface type='udp'>
      <mac address='44:38:39:00:01:00'/>
      <source address='172.20.232.71' port='33701'>
        <local address='172.20.232.22' port='33951'/>
      </source>
      <target dev='eth1'/>
      <model type='virtio'/>
      <alias name='net1'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x02' function='0x0'/>
    </interface>
</devices>
...
```
```
virsh shutdown netq1 && virsh start netq1
```

#### Verify UDP tunnel
##### NetQ

Add interfaces in netplan then restart netplan.
```
cat /etc/netplan/01-eth0.yaml
network:
    ethernets:
        eth0:
            dhcp4: true
            dhcp-identifier: mac
        eth1:
            dhcp4: true
            dhcp-identifier: mac
    version: 2
netplan apply -v
```
Verify eth1 is up. It should receive address from oob-mgmt-switch
```

3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 44:38:39:00:01:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.13/24 brd 192.168.200.255 scope global dynamic eth1
       valid_lft 172601sec preferred_lft 172601sec
    inet6 fe80::4638:39ff:fe00:100/64 scope link
       valid_lft forever preferred_lft forever
```

##### oob-mgmt-switch
```
root@oob-mgmt-switch:~# net sho lldp swp27
-------------------------------------------------------------------------------
LLDP neighbors:
-------------------------------------------------------------------------------
Interface:    swp27, via: LLDP, RID: 19, Time: 0 day, 00:04:14
  Chassis:
    ChassisID:    mac 52:54:00:bb:97:e0
    SysName:      netq1.bos1.rakops.com
    SysDescr:     Ubuntu 18.04.3 LTS Linux 4.15.0-72-generic #81-Ubuntu SMP Tue Nov 26 12:20:02 UTC 2019 x86_64
    MgmtIP:       172.20.232.68
    MgmtIP:       fe80::5054:ff:febb:97e0
    Capability:   Bridge, off
    Capability:   Router, off
    Capability:   Wlan, off
    Capability:   Station, on
  Port:
    PortID:       mac 44:38:39:00:01:00
    PortDescr:    eth1
    TTL:          120
-------------------------------------------------------------------------------

#### Install NetQ-agnet
Put the servers in a list. Install vagrant-scp plugin, add NetQ repo and install netq-agent.
```
vagrant plugin in vagrant-scp
for i in `cat servers.lst`;
  do vagrant scp netq.yml $i:/tmp/ \
     vagrant ssh -c 'grep netq /etc/hosts || echo "172.20.232.68 netq1.bos1.rakops.com" | sudo tee --append /etc/hosts && \
     grep netq /etc/apt/sources.list || echo "deb http://apps3.cumulusnetworks.com/repos/deb CumulusLinux-3 netq-latest" | sudo tee --append /etc/apt/sources.list && \
     apt-get update && apt-get install netq-agent -y && \
     sudo mv /tmp/netq.yml /etc/netq/config.d/netq.yml && \
     systemctl restart netq-agent' $i;
done

```

## References
1. [UDP Unicast Tunnel](https://libvirt.org/formatdomain.html#elementsNICSUDP)
2. [NetQ Deployment Guide](https://docs.cumulusnetworks.com/cumulus-netq/Cumulus-NetQ-Deployment-Guide/Install-NetQ/Prepare-NetQ-Onprem/#kvm-single-server-deployment)