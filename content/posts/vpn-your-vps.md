---
title: Configure & Setup Wireguard on
date: 2025-01-18
draft: false
tags:
  - vpn
  - vps
  - wireguard
---
## Step 1: Choose a VPS Provider and Set Up WireGuard Server

1. Create a VPS instance: Log in to your VPS provider and create a new instance. Choose an appropriate server size and location. I chose Digital Ocean.

2. Access your VPS: Use SSH to log in to your VPS from your local machine.
```
ssh user@your_vps_ip_address
```

## Step 2: Install WireGuard

The installation process varies slightly depending on your Linux distribution.
### 1. Ubuntu/Debian
```
sudo apt install wireguard
```
### 2. CentOS/Fedora:
```
sudo yum install epel-release
sudo yum install wireguard-tools
```

You may also need to install additional tools like 'qrencode' for generating QR codes and 'resolvconf' for DNS resolution.

## Step 3: Configure WireGuard

### **1. Generate Keys**
WireGuard uses public and private keys for encryption. Generate these keys on your VPS.
```
mkdir wireguard
cd wireguard
wg genkey | tee privatekey | wg pubkey > publickey
```

'privatekey': Your private key.  
'publickey': Your public key.

### **2. Create a Configuration File**
Create a configuration file for your WireGuard interface, typically located at '/etc/wireguard/wg0.conf'.
```
sudo nano /etc/wireguard/wg0.conf
```

```
[Interface]
PrivateKey = your_private_key
Address = 10.0.0.1/24  # IP range for VPN clients
ListenPort = 51820  # Default WireGuard port
[Peer]
PublicKey = client_public_key
AllowedIPs = 10.0.0.2/32  # Client IP within the VPN range
```

```
sudo ufw allow 51820/udp
net.ipv4.ip_forward=1
```

```
sudo wg-quick up/down wg0
```

sudo wg-quick up wg0

# Client

```
root@dest:/etc/wireguard# tail *
==> privatekey <==
IHelZ7GZzxsQl3Ka8E+QEkBjK5LcxNh7quUxL5A0FU8=

==> publickey <==
ePgp65HjlweruQRJKXiZslPKw96GaBv4As2JttyykSo=

==> wg0.conf <==
[Interface]
PrivateKey = IHelZ7GZzxsQl3Ka8E+QEkBjK5LcxNh7quUxL5A0FU8=
Address = 10.0.0.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = DVg14SzLUj3vysNVOiitjqAdIrgi6dA/ZClFcwLufmQ=
Endpoint = 188.166.34.118:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25

```
# Server

```
==> privatekey <==
MJxYQHvvCrjHvJ0cKyw1D2QSFrR+7BbyDjmJlY97LF4=

==> publickey <==
DVg14SzLUj3vysNVOiitjqAdIrgi6dA/ZClFcwLufmQ=

==> wg0.conf <==
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
PublicKey = ePgp65HjlweruQRJKXiZslPKw96GaBv4As2JttyykSo=
AllowedIPs = 10.0.0.2/32

```

Run on server 
```
iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables-save > /etc/iptables/rules.v4
```