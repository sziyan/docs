---
tags:
   - openshift
title: OpenShift Local
---


!!!info "Reference"
      - https://github.com/LibVNC/x11vnc
      - https://unix.stackexchange.com/questions/402201/creating-x11vnc-system-service

## Installation
1. Login to [OpenShift Hybrid Cloud Console](https://console.redhat.com/openshift/downloads) → Clusters → Create Cluster → OpenShift Local
2. Download Linux version
3. Create a VM with supported Linux OS such as Fedora
4. Ensure VM support virtualization (e.g in Proxmox, enable Host CPU mode)
5. Copy the package downloaded from step 2 into the VM.
6. Configure RAM and CPU
   `crc config set memory 16000`
   `crc config set cpus 4`
7. Install crc
   `crc setup`
8. Once the installation and dependencies are installed, we can start OpenShift Local
   `crc start`
9. kubeadmin and developer credentials are now shown on screen.

## Remote Access

1. Install HAProxy
   `sudo dnf install haproxy`

2. Allow traffic through firewall
```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --add-service=kube-apiserver --permanent
sudo firewall-cmd --reload
```

3. Allow API port 6443 through SELinux
   `sudo semanage port -a -t http_port_t -p tcp 6443`

4. Backup HAProxy configuration
   `sudo cp /etc/haproxy.cfg /etc/haproxy.cfg.backup`

5. Configure HAProxy

6. Get the IP of the CRC container
   ```
   crc ip
   ```
   ```cfg title="/etc/haproxy.cfg"
   global
      log /dev/log local0

   defaults
      balance roundrobin
      log global
      maxconn 100
      mode tcp
      timeout connect 5s
      timeout client 500s
      timeout server 500s

   listen apps
      bind 0.0.0.0:80
      server crcvm $CRC_IP:80 check

   listen apps_ssl
      bind 0.0.0.0:443
      server crcvm $CRC_IP:443 check

   listen api
      bind 0.0.0.0:6443
      server crcvm $CRC_IP:6443 check
   ```

7. Start HAProxy service
   ```bash
   sudo systemctl enable haproxy --now
   sudo systemctl start haproxy
   ```

8. Change DNS entry to point to the IP address of the VM.

9. To get kubeadmin credentials, run `crc console --credentials`
