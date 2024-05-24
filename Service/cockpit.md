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

refer to https://thiscute.world/posts/qemu-kvm-usage/
comment out these lines in /etc/libvirt/libvirtd.conf
```conf
unix_sock_rw_perms = "0770"

unix_sock_admin_perms = "0700"

unix_sock_dir = "/run/libvirt"
```

```bash
sudo systemctl restart libvirtd.service
```

## VM
```bash
virsh list
virsh list --all


virsh start ubuntu-24.04-desktop-amd64

virsh pool-list --all

virsh net-list --all
```

### 快照
```bash
virsh snapshot-create-as ubuntu-24.04-desktop-amd64 init
virsh snapshot-list ubuntu-24.04-desktop-amd64

virsh snapshot-revert ubuntu-24.04-desktop-amd64 init
virsh snapshot-delete ubuntu-24.04-desktop-amd64 init
```

## 参考
[^cockpit]: https://cockpit-project.org/documentation.html




