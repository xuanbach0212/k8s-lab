# OS upgrades

If the node was down for more than 5 minutes, then the pods are terminated from that node (base on taints & toleration NoExecute)

Default tolerations on every pod (injected by default mutating admission):

tolerations:

- key: node.kubernetes.io/not-ready
  effect: NoExecute
  tolerationSeconds: 300 # ← 5 min before eviction
- key: node.kubernetes.io/unreachable
  effect: NoExecute
  tolerationSeconds: 300 # ← 5 min before eviction

### cmd

- Draining a Node (gracefully shutdown pods -> recreate on other nodes. at the same time marked as unschedulable)

kubectl drain node-1

- Mark a node unschedulable

kubectl cordon node-1

- Mark a node schedulable

kubectl uncordon node-1
