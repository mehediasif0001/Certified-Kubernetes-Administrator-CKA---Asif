```
k get po  --all-namespaces | grep -i controlplane
```

What is the path of the directory holding the static pod definition files?
ans;

Run the command
```
ps -aux | grep kubelet
```
 and identify the config file 
 --config=/var/lib/kubelet/config.yaml. Then check in the config file for staticPodPath.

 ```
controlplane ~ ➜  grep -i staticpod /var/lib/kubelet/config.yaml

staticPodPath: /etc/kubernetes/manifests
```

Create a static pod named static-busybox that uses the busybox image , run in the default namespace and the command sleep 1000

ans;
To manage a static pod, you must edit, add, or remove the manifest file in the designated directory.

Create a pod definition file called static-busybox.yaml with the provided specs and place it under /etc/kubernetes/manifests directory.
```
controlplane /etc/kubernetes/manifests ➜  ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

controlplane /etc/kubernetes/manifests ➜  k run --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml

controlplane /etc/kubernetes/manifests ➜  ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml  static-busybox.yaml

controlplane /etc/kubernetes/manifests ➜  k get po
NAME                          READY   STATUS    RESTARTS   AGE
static-busybox-controlplane   1/1     Running   0          14s


```

We just created a new static pod named static-greenbox. Find it and delete it.


This question is a bit tricky. But if you use the knowledge you gained in the previous questions in this lab, you should be able to find the answer to it.

While trying to SSH to node01, if you are prompted for a password, use newRootP@ssw0rd.

ans:

First, let's identify the node in which the pod called static-greenbox is created. To do this, run:
```
root@controlplane:~# kubectl get pods --all-namespaces -o wide  | grep static-greenbox
default       static-greenbox-node01                 1/1     Running   0          19s     10.244.1.2   node01       <none>           <none>
root@controlplane:~#
From the result of this command, we can see that the pod is running on node01.

```
Next, SSH to node01 and identify the path configured for static pods in this node.

Important: The path need not be /etc/kubernetes/manifests. Make sure to check the path configured in the kubelet configuration file.
```
root@controlplane:~# ssh node01 
root@node01:~# ps -aux | grep kubelet

root       22184  0.0  0.1 2913688 78464 ?       Ssl  13:09   0:05 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --pod-infra-container-image=registry.k8s.io/pause:3.10.1
root       27354  0.0  0.0   6800  2448 pts/0    S+   13:17   0:00 grep kubelet

root@node01:~#grep -i staticpod /var/lib/kubelet/config.yaml
staticPodPath: /etc/just-to-mess-with-you
```
root@node01:~# 
Here the staticPodPath is /etc/just-to-mess-with-you


Navigate to this directory and delete the YAML file:
```
root@node01: cd /etc/just-to-mess-with-you
root@node01:/etc/just-to-mess-with-you# ls
greenbox.yaml
root@node01:/etc/just-to-mess-with-you# rm -rf greenbox.yaml 
root@node01:/etc/just-to-mess-with-you#
```
wait for 30 seconds.

Exit out of node01 using CTRL + D or type exit. You should return to the controlplane node, Check if the static-greenbox pod has been deleted:
root@controlplane:~# kubectl get pods --all-namespaces -o wide  | grep static-greenbox
root@controlplane:~# 

