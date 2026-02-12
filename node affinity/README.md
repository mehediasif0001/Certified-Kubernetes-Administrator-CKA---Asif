```
controlplane ~ ➜  k create deploy blu-deploy --image=nginx --dry-run=client -o yaml > dep
loy.yaml
```
```
vi deploy.yml
```
```
controlplane ~ ➜  k get node 
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   25m   v1.34.0
node01         Ready    <none>          25m   v1.34.

controlplane ~ ➜  k label node controlplane  nodename=asif
node/controlplane labeled
```

```
controlplane ~ ➜  k apply -f deploy.yaml 
deployment.apps/blu-deploy created

controlplane ~ ➜  k get deploy
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
blu-deploy   1/1     1            1           8s

controlplane ~ ➜  k get po -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blu-deploy-77cbcbc686-cjtnm   1/1     Running   0          22s   172.17.0.8   controlplane   <none>           <none>

```

