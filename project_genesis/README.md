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