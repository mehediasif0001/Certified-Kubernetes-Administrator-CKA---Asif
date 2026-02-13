
```
root@controlplane ~ ➜  k get priorityclasses
NAME                      VALUE        GLOBAL-DEFAULT   AGE     PREEMPTIONPOLICY
system-cluster-critical   2000000000   false            6m36s   PreemptLowerPriority
system-node-critical      2000001000   false            6m36s   PreemptLowerPriority
```
Create a PriorityClass named high-priority, a value of 100000, and a preemption policy of PreemptLowerPriority. Do not set this class as a global default.

```
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "This is priority class conf file  "
```

```
root@controlplane ~ ➜  k apply -f priorityClasses.yml 
priorityclass.scheduling.k8s.io/high-priority created

root@controlplane ~ ➜  k get priorityclasses
NAME                      VALUE        GLOBAL-DEFAULT   AGE   PREEMPTIONPOLICY
high-priority             100000       false            14s   PreemptLowerPriority
system-cluster-critical   2000000000   false            12m   PreemptLowerPriority
system-node-critical      2000001000   false            12m   PreemptLowerPriority

root@controlplane ~ ➜  
```

Create another PriorityClass named low-priority with a value of 1000. Do not set this class as a global default.

```
root@controlplane ~ ➜  cp priorityClasses.yml ./lower_priorityclasses.yml

root@controlplane ~ ➜  vi lower_priorityclasses.yml
```
```
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 1000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "This is lower priority class conf file  "
```
```
root@controlplane ~ ➜  k apply -f lower_priorityclasses.yml 
priorityclass.scheduling.k8s.io/low-priority created

root@controlplane ~ ➜  k get priorityclasses
NAME                      VALUE        GLOBAL-DEFAULT   AGE    PREEMPTIONPOLICY
high-priority             100000       false            4m5s   PreemptLowerPriority
low-priority              1000         false            64s    PreemptLowerPriority
system-cluster-critical   2000000000   false            16m    PreemptLowerPriority
system-node-critical      2000001000   false            16m    PreemptLowerPriority

```

In the default namespace, create a pod named low-prio-pod that runs an nginx image and uses the low-priority PriorityClass.

```
root@controlplane ~ ➜  k run --image=nginx low-prio-pod  --dry-run=client -o yaml > low-prio-pod.yml

root@controlplane ~ ➜  vi low-prio-pod.yml
```
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: low-prio-pod
  name: low-prio-pod
spec:
  containers:
  - image: nginx
    name: low-prio-pod
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  priorityClassName: low-priority
```

```                         
root@controlplane ~ ➜  k apply -f low-prio-pod.yml 
pod/low-prio-pod created


root@controlplane ~ ➜  k get po
NAME           READY   STATUS    RESTARTS   AGE
low-prio-pod   1/1     Running   0          12s

root@controlplane ~ ➜  
```

Now, create another pod in the default namespace named high-prio-pod that runs an nginx image and uses the high-priority PriorityClass.

```
root@controlplane ~ ➜   k run --image=nginx high-prio-pod  --dry-run=client -o yaml > high-prio-pod.yml

root@controlplane ~ ➜  vi high-prio-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: high-prio-pod
  name: high-prio-pod
spec:
  containers:
  - image: nginx
    name: high-prio-pod
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  priorityClassName: high-priority
```

```
root@controlplane ~ ➜  k apply -f high-prio-pod.yml 
pod/high-prio-pod created

root@controlplane ~ ➜  k get po
NAME            READY   STATUS    RESTARTS   AGE
high-prio-pod   1/1     Running   0          2s
low-prio-pod    1/1     Running   0          3m16s

```

You can compare the priority classes on both pods using the following command:

```
root@controlplane ~ ➜  kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"
NAME            PRIORITY
high-prio-pod   high-priority
low-prio-pod    low-priority

```
```
root@controlplane ~ ➜  k get po
NAME            READY   STATUS    RESTARTS   AGE
critical-app    0/1     Pending   0          23s
high-prio-pod   1/1     Running   0          97s
low-app         1/1     Running   0          23s
low-prio-pod    1/1     Running   0          4m51s

root@controlplane ~ ➜
```

The pod critical-app is stuck in Pending state due to the low-app pod getting scheduled and requesting high resources.

Check the status of the critical-app pod using the following command:
```
kubectl describe pod critical-app
```
massage:
```
0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory. no new claims to deallocate, preemption: 0/1 nodes are available:
1 No preemption victims found for incoming pod.
```

To resolve this situation and get the critical-app to a running state, you should:

Assign the high-priority class to the critical-app
Delete and recreate the pod with the new priority

```
root@controlplane ~ ➜  k get po critical-app -o yaml > critical-app.yml

root@controlplane ~ ➜  vi critical-app.yml
```
```
# critical-app.yaml
apiVersion: v1
kind: Pod
metadata:
  ...
  name: critical-app
  ...
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: critical-container
    ...
  dnsPolicy: ClusterFirst
  priorityClassName: high-priority   # Add the high-priority class
  enableServiceLinks: true
  preemptionPolicy: PreemptLowerPriority
  priority: 0  # Remove this line as this is the old default priority

```

```
root@controlplane ~  k delete po critical-app
pod "critical-app" deleted from default namespace

root@controlplane ~ ➜  k apply -f critical-app.yml 
pod/critical-app created

root@controlplane ~ ➜  k get po
NAME            READY   STATUS    RESTARTS   AGE
critical-app    1/1     Running   0          4s
high-prio-pod   1/1     Running   0          10m
low-prio-pod    1/1     Running   0          13m
```
