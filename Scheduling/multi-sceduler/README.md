What is the name of the POD that deploys the default kubernetes scheduler in this environment?

```controlplane ~ ➜  k get po -n kube-system
controlplane ~ ➜  k get po -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-6678bcd974-bzdbh               1/1     Running   0          4m34s
coredns-6678bcd974-vl7kq               1/1     Running   0          4m34s
etcd-controlplane                      1/1     Running   0          4m40s
kube-apiserver-controlplane            1/1     Running   0          4m41s
kube-controller-manager-controlplane   1/1     Running   0          4m40s
kube-proxy-7cpsf                       1/1     Running   0          4m34s
kube-scheduler-controlplane            1/1     Running   0          4m40s <---- this is
```
to see in details 

```
k describe pod kube-scheduler-controlplane -n kube-system
```
