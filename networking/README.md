What is the MAC address assigned to node01?

```
controlplane ~ ➜  k get node -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
controlplane   Ready    control-plane   27m   v1.34.0   192.168.42.128    <none>        Ubuntu 22.04.5 LTS   5.15.0-1083-gcp   containerd://1.6.26
node01         Ready    <none>          26m   v1.34.0   192.168.212.157   <none>        Ubuntu 22.04.5 LTS   5.15.0-1083-gcp   containerd://1.6.26
```

```
controlplane ~ ➜  ssh node01
node01 ~ ➜  ip a | grep -B2 192.168.212.157 
4: eth0@if28130: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether a6:d8:8a:4c:7c:fc brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.212.157/32 scope global eth0
```

We use Containerd as our container runtime. What is the interface/bridge created by Containerd on the controlplane node?

```
controlplane ~ ➜  ip a show type bridge

6: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UP group default qlen 1000
    link/ether 5e:70:1d:56:8a:06 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/24 brd 172.17.0.255 scope global cni0
       valid_lft forever preferred_lft forever
    inet6 fe80::5c70:1dff:fe56:8a06/64 scope link 
       valid_lft forever preferred_lft forever
```
What is the state of the interface cni0?

```
controlplane ~ ➜  ip a show type bridge | grep state
6: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UP group default qlen 1000
```

If you were to ping google from the controlplane node, which route does it take?

```
controlplane ~ ➜  route 
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         169.254.1.1     0.0.0.0         UG    0      0        0 eth0  <<<<<< this is default gateway
169.254.1.1     0.0.0.0         255.255.255.255 UH    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.255.0   U     0      0        0 cni0
172.17.1.0      172.17.1.0      255.255.255.0   UG    0      0        0 flannel.1
```
ans: 169.254.1.1 


What is the port the kube-scheduler is listening on in the controlplane node?
```
controlplane ~ ➜  netstat -tulp | grep kube-scheduler
tcp        0      0 localhost:10259         0.0.0.0:*               LISTEN      2880/kube-scheduler 
```
ans: 10259

Notice that ETCD is listening on two ports. Which of these have more client connections established?

use this command 
```
netstat -anp | grep -i etcd
```
 -p, --programs           display PID/Program name for sockets

 
 -n, --numeric            don't resolve names


 
 -a, --all                display all sockets (default: connected)

 


 then grep port 

 ```
controlplane ~ ➜  netstat -anp | grep -i etcd | grep 2379 | wc -l
64

controlplane ~ ➜  netstat -anp | grep -i etcd | grep 2381 | wc -l
1
```

ans : 64



 Inspect the kubelet service and identify the container runtime endpoint value is set for Kubernetes.

 ```
controlplane ~ ➜  cat /var/lib/kubelet/config.yaml | grep -i endpoint
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
```
ans: unix:///var/run/containerd/containerd.sock



What is the path configured with all binaries of CNI supported plugins?

```
controlplane ~ ➜  ls /opt/cni/bin
bandwidth  dummy     host-device  LICENSE   portmap    sbr     tuning
bridge     firewall  host-local   loopback  ptp        static  vlan
dhcp       flannel   ipvlan       macvlan   README.md  tap     vrf
```
ans is : The CNI binaries are located under /opt/cni/bin by default.


What is the CNI plugin configured to be used on this kubernetes cluster?

ans: Run the command: ls /etc/cni/net.d/ and identify the name of the plugin.

```
controlplane ~ ➜  cd /etc/cni/net.d/

controlplane /etc/cni/net.d ➜  ls
10-flannel.conflist
```
```
controlplane /etc/cni/net.d ➜  cat 10-flannel.conflist 
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```





