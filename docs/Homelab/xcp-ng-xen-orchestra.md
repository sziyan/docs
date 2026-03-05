---
tags: 
   - xcp
   - xen orchestra
created: 2026-01-08
modified: 2026-01-12
title: XCP NG and Xen Orchestra
---

# XCP NG and Xen Orchestra

In order to install Xen Orchestra, a VM need to be created in XO Lite.

## Creating the VM
1. After XCP-NG is installed, navigate to the IP address of the host (XO Lite) from a web browser.
2. Login as root user created during installation of node
3. Create a VM, and boot into PXE
4. From netboot, select a Linux distro to install.
5. After installation is completed, shutdown the VM.
6. Login to XCP NG with root user via SSH
7. Get the UUID of the VM
   ```
   xe vm-list name-label=vm_name params=uuid
   ```
8. Change the boot order to boot from disk first, then network:
   ```
   xe vm-param-set uuid=<from step 7> HVM-boot-params:order=cn
   ```
9. Go back to XO Lite and start the VM. The VM should now boot from disk
10. Set static IP address of the VM from the VM console.

## Installation of Xen Orchestra
1. SSH to the create VM and clone [XenOrchestraInstallerScript](https://github.com/ronivay/XenOrchestraInstallerUpdater)
2. Change directory to the cloned folder 
```
cd XenOrchestraInstallerUpdater
```
3. Copy the sample config file 
```
cp sample.xo-install.cfg xo-install.cfg
```
4. Not much configuration need to be done, but the [wiki](https://github.com/ronivay/XenOrchestraInstallerUpdater/wiki) show more information on the configuration parameters available.
5. Run the install script 
```
sudo ./xo-install.sh
```
6. Select `Install` and wait for the installation to complete.
7. Once installation is completed, navigate to the IP address set in step 10 [[#Creating the VM]] from a web browser

# Tips and configuration

## Increase API token max validity period

1. SSH to Xen Orchestra VM as user and change user to root.
2. Change directory to user config folder `~/.config/xo-server`
3. Create a new configuration file
```toml title="config.authentication.toml" hl_lines="3"
[authentication]
defaultTokenValidity = '30 days'
maxTokenValidity = '1 year'
```
4. Restart xo-server service
   `systemctl restart xo-server`

!!! hint "Reference"
    - [xen-orchestra/packages/xo-server/config.toml at 6c44a94bf4b7adc2d3a46d5a7813f6a675b8f49b · vatesfr/xen-orchestra · GitHub](https://github.com/vatesfr/xen-orchestra/blob/6c44a94bf4b7adc2d3a46d5a7813f6a675b8f49b/packages/xo-server/config.toml#L46)


## Creating cloudinit template

Xen Orchestra and XCP only support raw or vhd images. Thus, a “middle” VM is needed to convert to the correct format. 

1. Create a a VM of any Linux distro. Install qemu tools
   ```
   sudo apt install qemu-utils -y
   ```
2. Download the qcow2 cloud init images into the VM.
3. Convert to vhd format
   ```
   qemu-img convert -O vpc <downloaded_file_name>.qcow2 <downloaded_file_name>.vhd
   ```
4. Copy this file to XCP host via scp or rsync
   ```
   scp <downloaded_file_name>.vhd root@<xcp_ip>:/run/sr-mount/<local_storage_uuid>/
   ```
5. Create a VM with empty disk. Click Advanced Settings, and uncheck “Boot VM after creation”.
6. Make sure to disable network boot under Advanced tab after VM creation. 
7. In Xen Orchestra, navigate to Home > Storage > Local Storage, and click rescan all disk. Make sure the newly imported disk from step 4 is present. Rename the disk to something recognizable.
8. Go to Home > VM > name of VM created in step 6.
9. Go to Disk tab, and click Attach disk. Attach the disk from step 7.
10. Go to Advance tab > and click convert to template.