# Network Namespaces

- Isolate a process's network stack (interfaces, routing table, ARP table, iptables) from the host.
- This is how each container/Pod gets its own network — a Pod = one network namespace shared by all its containers.

## Create namespaces

```bash
ip netns add red
ip netns add blue
ip netns                              # list

ip netns exec red ip link             # run a command inside the namespace
ip -n red link                        # shorthand
```

## Connect 2 namespaces directly (veth pair)

A **veth pair** is a virtual cable — packets sent into one end come out the other.

```bash
ip link add veth-red type veth peer name veth-blue

ip link set veth-red netns red
ip link set veth-blue netns blue

ip -n red addr add 192.168.15.1/24 dev veth-red
ip -n blue addr add 192.168.15.2/24 dev veth-blue

ip -n red link set veth-red up
ip -n blue link set veth-blue up

ip netns exec red ping 192.168.15.2
```

## Connect many namespaces (Linux bridge)

Point-to-point veth pairs don't scale — use a virtual switch (bridge) on the host instead.

```bash
# 1. create the bridge (it's just another interface on the host)
ip link add v-net-0 type bridge
ip link set dev v-net-0 up

# 2. for each namespace: veth pair, one end in the ns, other end attached to the bridge
ip link add veth-red type veth peer name veth-red-br
ip link set veth-red netns red
ip link set veth-red-br master v-net-0

ip -n red addr add 192.168.15.1/24 dev veth-red
ip -n red link set veth-red up
ip link set veth-red-br up

# 3. give the bridge an IP so the host can reach the namespaces
ip addr add 192.168.15.5/24 dev v-net-0
```

## Reach the outside world

```bash
# route out via the bridge (host acts as the gateway)
ip netns exec red ip route add default via 192.168.15.5

# NAT: replace the namespace's source IP with the host's IP
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
```

## Reach the namespace from outside (port forwarding)

```bash
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.15.2:80
```

These exact steps (create ns → veth → attach to bridge → assign IP → NAT) are what a CNI plugin automates for every Pod — see [cni.md](cni.md).
