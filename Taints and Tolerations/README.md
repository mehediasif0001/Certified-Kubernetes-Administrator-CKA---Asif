```
controlplane ~ ➜  k describe node/controlplane  | grep -i taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```
```
controlplane ~ ➜  k taint node controlplane  node-role.kubernetes.io/control-plane:NoSchedule-
node/controlplane untainted
```
