# WiFi Bridge

I'm using this on a raspberry pi 3 running Ubuntu server to bridge an Ethernet-only OpenSprinker to my WiFi network.

## Install
```sh
sudo apt install parprouted dhcp-helper avahi-daemon net-tools
```

/etc/netplan/60-wifi-bridge.yaml:
```yaml
network:
  version: 2
  ethernets:
    renderer: networkd
    eth0:
      optional: true
      dhcp4: false
```

/etc/sysctl.d/local.conf:
```
net.ipv4.ip_forward=1
```


/etc/networkd-dispatcher/routable.d/50-wifi-bridge:
```sh
#!/bin/sh

if [ "$IFACE" = "wlan0" ]; then
  /sbin/ip link set wlan0 promisc on
  /sbin/ip addr add $(/sbin/ip addr show wlan0 | perl -wne 'm|^\s+inet (.*)/| && print $1')/32 dev eth0
  /usr/sbin/parprouted eth0 wlan0
fi
```

/etc/networkd-dispatcher/off.d/50-wifi-bridge:
```sh
#!/bin/sh

if [ "$IFACE" = "wlan0" ]; then
  /usr/bin/killall /usr/sbin/parprouted
  /sbin/ip link set eth0 down
fi
```

Edit /etc/default/dhcp-helper:
```sh
# relay dhcp requests as broadcast to wlan0
DHCPHELPER_OPTS="-b wlan0"
```

Edit /etc/avahi/avahi-daemon.conf:
```
[reflector]
enable-reflector=yes
```

Enable services:
```sh
sudo systemctl enable dhcp-helper --now
```

Reboot!

### References
- Main documentation: https://wiki.debian.org/BridgeNetworkConnectionsProxyArp
- Netplan yaml reference: https://netplan.readthedocs.io/en/stable/netplan-yaml/
- Netplan FAQ for post-up/post-down scripts: https://netplan.io/faq
- networkd-dispatcher docs: https://gitlab.com/craftyguy/networkd-dispatcher/-/tree/master
- WiFi promiscuous mode: https://github.com/PiSCSI/piscsi/issues/1387

Inspired by: https://gist.github.com/Jiab77/76000284f8200da5019a232854421564
