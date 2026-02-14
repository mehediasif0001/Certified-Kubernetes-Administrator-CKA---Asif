#node maintenance 

Let us explore the environment first. How many nodes do you see in the cluster?

```
controlplane ~ ➜  k get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   14m   v1.34.0
node01         Ready    <none>          13m   v1.34.0
```
How many applications do you see hosted on the cluster?

Check the number of deployments in the default namespace.

controlplane ~ ➜  kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
blue   3/3     3            3           51s

Which nodes are the applications hosted on?

```
controlplane ~ ➜  k get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
blue-759779556-d8sfq   1/1     Running   0          110s   172.17.1.3   node01         <none>           <none>
blue-759779556-nmj4w   1/1     Running   0          110s   172.17.1.2   node01         <none>           <none>
blue-759779556-wz6f7   1/1     Running   0          110s   172.17.0.4   controlplane   <none>           <none>
```

We need to take node01 out for maintenance. Empty the node of all applications and mark it unschedulable.

```
controlplane ~ ➜  k drain node01 --ignore-daemonsets
node/node01 already cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-c5ldn, kube-system/kube-proxy-jbn45
evicting pod default/blue-759779556-nmj4w
evicting pod default/blue-759779556-d8sfq
pod/blue-759779556-d8sfq evicted
pod/blue-759779556-nmj4w evicted
node/node01 drained

controlplane ~ ➜  k get po -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-759779556-2r7rz   1/1     Running   0          25s     172.17.0.6   controlplane   <none>           <none>
blue-759779556-79rls   1/1     Running   0          25s     172.17.0.5   controlplane   <none>           <none>
blue-759779556-wz6f7   1/1     Running   0          3m51s   172.17.0.4   controlplane   <none>           <none>
```

The maintenance tasks have been completed. Configure the node node01 to be schedulable again.

```
controlplane ~ ➜  k uncordon node01
node/node01 uncordoned
```

Running the uncordon command on a node will not automatically schedule pods on the node. When new pods are created, they will be placed on node01.

Why are the pods placed on the controlplane node?

```
root@controlplane:~# kubectl describe node controlplane | grep -i  taint
Taints:             <none>
```


last task : hr-app is a critical app and we do not want it to be removed and we do not want to schedule any more pods on node01.
Mark node01 as unschedulable so that no new pods are scheduled on this node.

 ```
controlplane ~ ➜  k cordon node01
node/node01 cordoned
```

Cluster Management Commands:
```


  cluster-info    Display cluster information
  top             Display resource (CPU/memory) usage
  cordon          Mark node as unschedulable
  uncordon        Mark node as schedulable
  drain           Drain node in preparation for maintenance
  taint           Update the taints on one or more nodes
```
