---
tags:
   - xcp
   - proxmox
   - hypervisor
   - homelab
created: 2026-01-10
modified: 2026-01-11
title: Migrate Proxmox VM to XCP-NG
---

# Migrate Proxmox VM to XCP-NG

Make sure Xen Orchestra is already installed by following the steps in [XCP NG and Xen Orchestra](xcp-ng-xen-orchestra.md)

## Export Proxmox VM disk

1. SSH to the Proxmox host that is running the VM.
2. Get the path of the disk 
   ```
   pvesm path vm:vm-100-disk-0
   ```
    where `100` is the VM ID and `vm:` is the storage in Proxmox.
3. Convert the disk to qcow2 format:
```
qemu-img convert -f raw -O qcow2 /dev/vm/vm-100-disk-0 `vm_name`.qcow2
```
4. Convert generated qcow2 file to VHD format:
   ```
   qemu-img convert -O vpc <vm_name>.qcow2  <vm_name>.vhd
   ```

## Import to XCP
1. Transfer the generated file to XCP host
   ```
   scp <file name>.vhd root@<xcp_ng_ip>:~/
   ```
2. Optional: Do a verification of the VHD file 
```
vhd-util repair -n </path/to/vhd_file>; vhd-util check -n </path/to/vhd_file>` . 
```
   The result should show the filename is valid.
3. Move the file to xcp local storage folder
```
mv <vm_name>.vhd /run/sr-mount/<local storage uuid>/`uuidgen`.vhd
```
To find out the storage uuid:
`cd /run/sr-mount` and copy the folder name
4. Navigate to Xen Orchestra from a web browser, and go to Storage tab > Local Storage > Scan Disk. The new disk should be reflected there.

!!! info "Reference"
    - [KVM qcow2 disk converted and and imported to xcp-ng just works? \| XCP-ng and XO forum](https://xcp-ng.org/forum/topic/8322/kvm-qcow2-disk-converted-and-and-imported-to-xcp-ng-just-works/2?_=1768053850844)
    - [Migrate to XCP-ng \| XCP-ng Documentation](https://docs.xcp-ng.org/installation/migrate-to-xcp-ng/#-from-kvm-libvirt)