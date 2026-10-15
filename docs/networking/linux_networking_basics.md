# Linux Networking Basics

Prerequisites for understanding how Pods talk to each other — k8s networking is built on these primitives.

## Switching and routing

- **Switch** — connects hosts on the **same** network.
- **Router** — connects **different** networks.
- **Gateway** — the route a host uses to reach another network. `default` (or `0.0.0.0`) = anything with no more specific route.

```bash
ip link                                   # list interfaces
ip addr                                   # list IPs on interfaces
ip addr add 192.168.1.10/24 dev eth0      # assign IP
ip route                                  # show routing table
ip route add 192.168.2.0/24 via 192.168.1.1
ip route add default via 192.168.1.1
```

- Changes made with `ip` are **not persistent** across reboots — persist them in `/etc/network/interfaces` or netplan.

## IP forwarding

A Linux host does **not** forward packets between its interfaces by default — required for any host acting as a router (including every k8s node).

```bash
cat /proc/sys/net/ipv4/ip_forward         # 0 = off, 1 = on

echo 1 > /proc/sys/net/ipv4/ip_forward    # temporary

# persistent: set in /etc/sysctl.conf (or /etc/sysctl.d/k8s.conf)
net.ipv4.ip_forward = 1
sysctl --system
```

## DNS

- `/etc/hosts` — local static name → IP mapping.
- `/etc/resolv.conf` — which DNS server to ask (`nameserver`) and which domain suffixes to try (`search`).
- `/etc/nsswitch.conf` — lookup order (`hosts: files dns` = `/etc/hosts` first, then DNS).

```bash
cat /etc/resolv.conf
# nameserver 192.168.1.100
# search mycompany.com

nslookup www.google.com   # queries DNS only, ignores /etc/hosts
dig www.google.com        # more detail
```

Record types:

- **A** — name → IPv4
- **AAAA** — name → IPv6
- **CNAME** — name → another name

## Useful commands

```bash
ip neighbor                 # ARP table
netstat -plnt               # listening TCP ports + process
ss -tlnp                    # same, newer tool
```
