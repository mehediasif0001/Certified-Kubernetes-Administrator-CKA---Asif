# argument in k8s 
```
kubectl run <pod-name> \
  --image=<image> \
  --dry-run=client \
  -o yaml \
  -- <args> > pod.yml

```

# Config map 

```
controlplane ~ ➜  k create cm  webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disreg
ard --dry-run=client -o yaml >   webapp-config-map.yaml

controlplane ~ ➜  k get cm
NAME               DATA   AGE
db-config          3      11m
kube-root-ca.crt   1      31m
```
```
apiVersion: v1
data:
  APP_COLOR: darkblue
  APP_OTHER: disregard
kind: ConfigMap
metadata:
  name: webapp-config-map
```
```
controlplane ~ ➜  k apply -f webapp-config-map.yaml 
configmap/webapp-config-map created

controlplane ~ ➜  k get cm 
NAME                DATA   AGE
db-config           3      11m
kube-root-ca.crt    1      31m
webapp-config-map   2      4s

controlplane ~ ➜  k describe cm webapp-config-map
```

# secret
```
controlplane ~ ➜  k create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
secret/db-secret created

controlplane ~ ➜  k get secrets
NAME              TYPE                                  DATA   AGE
dashboard-token   kubernetes.io/service-account-token   3      9m13s
db-secret         Opaque                                3      14s
```
