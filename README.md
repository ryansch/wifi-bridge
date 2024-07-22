# WiFi Bridge

Reference: https://wiki.debian.org/BridgeNetworkConnectionsProxyArp

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
