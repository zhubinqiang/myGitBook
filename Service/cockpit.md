# cockpit

cockpit[^cockpit] 是一个基于web的管理工具。对于初学者友好的服务。

## cockpit on Ubuntu 22.04

```bash
sudo apt install -y cockpit cockpit-machines
```

```bash
sudo usermod -a -G libvirt-qemu ${USER}
```

access from hostIP:9090

virtual machines:
  Storage pools
    Create storage pool

VM:
/etc/libvirt/qemu/ubuntu-24.04-desktop-amd64.xml

/var/lib/libvirt/images/ubuntu-24.04-desktop-amd64.qcow2

> qcow 以及 iso 放在home目录下 可能会读取不到，有permission denied的问题。

## 参考
[^cockpit]: https://cockpit-project.org/documentation.html




