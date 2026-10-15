# Pod Networking

k8s does not implement Pod networking itself — it defines the rules, and a CNI plugin ([cni.md](cni.md)) implements them.

## The k8s network model

1. Every Pod gets its **own unique IP** address.
2. Every Pod can reach every other Pod **on the same node** using that IP.
3. Every Pod can reach every other Pod **on other nodes** using that IP — **without NAT**.

## How it's done (conceptually)

On each node:

1. A bridge (e.g. `cni0`) with its own subnet — e.g. node1 `10.244.1.0/24`, node2 `10.244.2.0/24`.
2. When a Pod is created, the runtime calls the CNI plugin, which:
   - creates a veth pair, one end inside the Pod's network namespace, one end on the bridge,
   - assigns a free IP from the node's subnet,
   - sets the default route inside the Pod to the bridge IP.

Across nodes, packets for `10.244.2.0/24` must get to node2. Two approaches:

- **Routing** — each node (or the router) has a route `10.244.2.0/24 via <node2-ip>` (Calico BGP does this).
- **Overlay** — the packet is encapsulated (VXLAN / IP-in-IP) in a node-to-node packet and unwrapped on the other side (Flannel VXLAN does this).

## Inspect

```bash
kubectl get pods -o wide                     # Pod IP + node
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.podCIDR}{"\n"}{end}'

# on a node
ip route                                     # routes to other nodes' Pod subnets
ip addr show cni0

# inside a Pod
kubectl exec -it <pod> -- ip addr
kubectl exec -it <pod> -- ip route
```
