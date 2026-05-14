---
title: "Configuring an Embedded Linux Machine as a WiFi Hotspot"
date: 2014-06-13T11:55:43+08:00
tags: ["linux", "embedded", "wifi", "networking", "systemd", "toradex"]
---

How to configure an embedded Linux board (Colibri VF61) as a WiFi access point using `hostapd`, `udhcpd`, and `iptables`.

## Test Setup

- Colibri VF61 V1.1A
- Colibri Evaluation Board V3.1a / Iris V1.1
- Ambicom WL250N-USB (Ralink RT3070 chipset)

## Steps Overview

1. Assign a static IP to `wlan0`
2. Configure `udhcpd` as the DHCP server
3. Configure `hostapd` as the AP daemon
4. Enable IP forwarding and NAT between `wlan0` and `eth0`
5. Wire it all up with systemd service files

## 1. Static IP for wlan0

```bash
ifconfig wlan0 192.168.0.1 up
```

## 2. udhcpd configuration

`/etc/udhcpd.conf`:
```
start           192.168.0.20
end             192.168.0.254
interface       wlan0
remaining       yes
lease_file      /var/lib/misc/udhcpd.leases
pidfile         /var/run/udhcpd.pid
opt dns         8.8.8.8 8.8.4.4
opt router      192.168.0.1
opt subnet      255.255.255.0
opt lease       864000
```

## 3. hostapd configuration

`/etc/hostapd.conf`:
```ini
interface=wlan0
driver=nl80211
ssid=MyEmbeddedAP
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_passphrase=your-passphrase
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

## 4. IP forwarding and NAT

```bash
# Enable forwarding
sysctl -w net.ipv4.ip_forward=1
# Uncomment net.ipv4.ip_forward=1 in /etc/sysctl.conf to persist

# NAT rule
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

# Save
iptables-save > /etc/iptables.ipv4.nat
```

## 5. systemd service

`/etc/systemd/system/access-point.service`:
```ini
[Unit]
Description=WiFi Access Point
BindsTo=sys-subsystem-net-devices-wlan0.device
After=sys-subsystem-net-devices-wlan0.device

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/bin/ip link set dev wlan0 up
ExecStart=/bin/ip addr add 192.168.0.1/24 broadcast 192.168.0.255 dev wlan0
ExecStart=/usr/sbin/hostapd -B /etc/hostapd.conf
ExecStart=/usr/sbin/udhcpd -fS /etc/udhcpd.conf
ExecStop=/sbin/ip addr flush dev wlan0
ExecStop=/sbin/ip link set dev wlan0 down

[Install]
WantedBy=multi-user.target
```

```bash
systemctl enable access-point.service
systemctl enable iptables-restore.service
```
