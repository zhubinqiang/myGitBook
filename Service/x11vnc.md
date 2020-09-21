# x11vnc
[TOC]

## CentOS 7
### Install
```bash
yum install -y x11vnc
```

### create password for x11vnc
```bash
x11vnc -storepasswd
sudo mv ~/.vnc/passwd /etc/x11vnc.pwd
```

### create the service
/etc/systemd/system/x11vnc.service
```
[Unit]
Description=Remote desktop service (VNC)
Requires=display-manager.service
After=display-manager.service

[Service]
Type=forking
ExecStart=/usr/bin/x11vnc -display :0 -forever -shared -bg -rfbauth /etc/x11vnc.pwd -o /var/log/x11vnc.log
ExecStop=/usr/bin/killall x11vnc
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```


```
# The vncserver service unit file
#
# Quick HowTo:
# 1. Copy this file to /etc/systemd/system/vncserver@.service
# 2. Edit <USER> and vncserver parameters appropriately
#   ("runuser -l <USER> -c /usr/bin/vncserver %i -arg1 -arg2")
# 3. Run `systemctl daemon-reload`
# 4. Run `systemctl enable vncserver@:<display>.service`
#
# DO NOT RUN THIS SERVICE if your local area network is
# untrusted!  For a secure way of using VNC, you should
# limit connections to the local host and then tunnel from
# the machine you want to view VNC on (host A) to the machine
# whose VNC output you want to view (host B)
#
# [user@hostA ~]$ ssh -v -C -L 590N:localhost:590M hostB
#
# this will open a connection on port 590N of your hostA to hostB's port 590M
# (in fact, it ssh-connects to hostB and then connects to localhost (on hostB).
# See the ssh man page for details on port forwarding)
#
# You can then point a VNC client on hostA at vncdisplay N of localhost and with
# the help of ssh, you end up seeing what hostB makes available on port 590M
#
# Use "-nolisten tcp" to prevent X connections to your VNC server via TCP.
#
# Use "-localhost" to prevent remote VNC clients connecting except when
# doing so through a secure tunnel.  See the "-via" option in the
# `man vncviewer' manual page.


[Unit]
Description=Remote desktop service (VNC)
#After=syslog.target network.target
Requires=display-manager.service
After=display-manager.service

[Service]
Type=forking
# Clean any existing files in /tmp/.X11-unix environment
# ExecStartPre=/bin/sh -c '/usr/bin/x11vnc -kill %i > /dev/null 2>&1 || :'
# ExecStart=/usr/sbin/runuser -l media -c "/usr/bin/x11vnc -display %i -forever -shared -bg -rfbauth /etc/x11vnc.pwd -o /var/log/x11vnc.log"
ExecStart=/usr/local/bin/x11vnc -display :0 -forever -shared -bg -rfbauth /etc/x11vnc.pwd -o /var/log/x11vnc.log
# PIDFile=/home/media/.vnc/%H%i.pid
# ExecStop=/bin/sh -c '/usr/bin/x11vnc -kill %i > /dev/null 2>&1 || :'
# ExecStop=/usr/bin/killall x11vnc
User=%I
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```


### enable service
```bash
sudo mv x11vnc.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable x11vnc.service
sudo systemctl start x11vnc.service
```

## Ubuntu 14.04
### Install
```bash
sudo apt-get install -y x11vnc
```

### config
/etc/init/x11vnc.conf
```
start on login-session-start
script
    x11vnc -display :0 -auth /var/run/lightdm/root/:0 -forever -bg -o /var/log/x11vnc.log -rfbport 5900
end script
```

## Ubuntu 17.04
```sh
sudo x11vnc -storepasswd /etc/x11vnc.pass
```

/lib/systemd/system/x11vnc.service
```
[Unit]
Description=Start x11vnc at startup.
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /etc/x11vnc.pass -rfbport 5900 -shared -o /var/log/x11vnc.log

[Install]
WantedBy=multi-user.target
```

```sh
sudo systemctl daemon-reload
sudo systemctl enable x11vnc.service
sudo systemctl start x11vnc.service
```

## SUSE12
### install
```bash
## add repo
zypper ar http://mirrors.aliyun.com/opensuse/distribution/12.2/repo/oss/ suse12.2

## instll x11vnc
zypper in -y x11vnc
```

