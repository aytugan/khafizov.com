# Wireguard

### Links

<https://www.wireguard.com/netns/#the-new-namespace-solution>

<https://www.opennet.ru/tips/2683_linux_namespace_gateway_virtual_route_iproute.shtml>

<https://www.stableit.ru/2015/06/bird-bgp.html>

<https://vincent.bernat.ch/en/blog/2018-route-based-vpn-wireguard>

Kilo is a multi-cloud network overlay built on WireGuard and designed for Kubernetes - <https://github.com/squat/kilo>


### Unofficial documentation

- <https://github.com/pirate/wireguard-docs>
- <https://monadical.com/posts/wireguard.html> (very detailed doc)

### Notes

> VPN channel should have same MTU from both sides

### Route

```bash
ip -4 route add 172.16.0.0/16 dev wg3

```

### Custom routing for tunnels

For custom routing, built-in wireguard routing should be switched off via `Table = off` and AllowedIPs should be configured as `AllowedIPs = 0.0.0.0/1, 128.0.0.0/1`

```
[Interface]
Address =  172.16.101.10/30
PrivateKey = *****=
ListenPort = 1234
Table = off

[Peer]
PublicKey = ****=
AllowedIPs = 0.0.0.0/1, 128.0.0.0/1
Endpoint = 11.22.33.44:1234
PersistentKeepalive = 25
```

### Autostart tunnel

```bash
systemctl enable wg-quick@wg1.service
```

### Hot tunnel reload

```bash
wg syncconf wg1 <(wg-quick strip wg1)
```


### Wireguard on windows as a service

- <https://r-pufky.github.io/docs/services/wireguard/windows-setup.html>

### Hooks for wireguard

```bash
PreUp = iptables -A INPUT -p udp --dport 5502 -j ACCEPT -m comment --comment "WG 5502 UDP"
PostUp = iptables -A INPUT -i wg5502 -j ACCEPT -m comment --comment "WG 5502 Tunnel"
PostDown = iptables -D INPUT -i wg5502 -j ACCEPT -m comment --comment "WG 5502 Tunnel"; iptables -D INPUT -p udp --dport 5502 -j ACCEPT -m comment --comment "WG 5502 UDP"
```

### Wireguard can be configured via netplan

 - <https://netplan.io/reference/>
 - <https://forum.turris.cz/t/wireguard-setup/6991/41>


```bash
tunnels:
  wg0:
    mode: wireguard
    addresses: [...]
    peers:
      - keys:
          public: rlbInAj0qV69CysWPQY7KEBnKxpYCpaWqOs/dLevdWc=
          shared: /path/to/shared.key
        ...
    key: mNb7OIIXTdgW4khM7OFlzJ+UPs7lmcWHV7xjPgakMkQ=
```

### Wireguard GUI

- Multi-user GUI - <https://docs.firezone.dev/docs/deploy/server/>
- Simple single-user GUI - <https://github.com/WeeJeWel/wg-easy>

### Simple VPN server installation

1) Deploy VPS with Ubuntu 22.04. Minimum 1 core and 0.5Gb RAM is required.

2) Login into VPS via ssh and run `sudo su -` command

3) Copy and paste below commands, enter "Y" or "Enter" on all requests

```bash
# Update system

apt -y update
apt -y install nano
apt -y dist-upgrade
apt -y autoremove

# Install fail2ban to prevent ssh password brute-force
apt -y install fail2ban

# Install docker

curl -sSL https://get.docker.com | sudo sh

# Install wg-easy
mkdir -p /opt/wgeasy
cd /opt/wgeasy
```

3) run `nano /opt/wgeasy/docker-compose.yml`

change `WG_HOST=11.22.33.44` to VPS IP address, `PASSWORD=changeme` to some password 
and paste into editor 

```docker
version: "3.8"
services:
  wg-easy:
    environment:
      # Change this to your host's public address
      - WG_HOST=11.22.33.44
      - PASSWORD=changeme
      - WG_DEFAULT_DNS=1.1.1.1,8.8.8.8
      - WG_MTU=1420

    image: weejewel/wg-easy
    container_name: wg-easy
    volumes:
      - .:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
```

4) Press `Ctrl-X`, `Y` and `Enter` to save a file

5) run `docker compose up -d`

6) open <http://your-vps-ip:51821> in browser to configure VPN server
