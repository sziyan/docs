---
title: Install VNC Server in Linux
tags:
    - linux
---

# Install VNC Server in Linux

1. Install x11vnc
```
sudo apt install x11vnc
```

2. Create a x11vnc password
```bash
x11vnc -storepasswd

Enter VNC Password:
Verify password:
Write password to /home/user/.vnc/passwd? [y]/n y
Password written to: /home/user/.vnc/passwd
```

3. Create a service (replace username and group with the user that created x11vnc passwd in step 2)
``` title="/etc/systemd/system/x11vnc.service" hl_lines="7 8"
[Unit]
Description=VNC Server for X11
After=multi-user.target

[Service]
ExecStart=/usr/bin/x11vnc -find -shared -usepw -forever
User=sziyan
Group=sziyan

[Install]
WantedBy=multi-user.target
```

4. Enable and start the service
```bash
sudo systemctl enable x11vnc
sudo systemctl start x11vnc
```

!!!info "Reference"
    - https://github.com/LibVNC/x11vnc
    - https://unix.stackexchange.com/questions/402201/creating-x11vnc-system-service