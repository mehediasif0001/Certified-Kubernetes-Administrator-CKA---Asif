An easy way to create a DaemonSet is to start by generating a Deployment YAML using:
```
kubectl create deployment elasticsearch
--image=registry.k8s.io/fluentd-elasticsearch:1.20
-n kube-system --dry-run=client -o yaml > fluentd.yaml
```
Then, edit the YAML as follows:

Change: 
kind: Deployment → kind: DaemonSet
Ensure: apiVersion: apps/v1
Remove these fields from the YAML:
spec.replicas
spec.strategy
status (entire section, if present)
Finally, create the DaemonSet using:
```
kubectl create -f fluentd.yaml
```
