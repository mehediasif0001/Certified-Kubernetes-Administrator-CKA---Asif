What is the Common Name (CN) configured on the Kube API Server Certificate?


Run the command 
```
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
```
and look for Subject CN.

task2:
 
Create a CertificateSigningRequest object with the name akshay with the contents of the akshay.csr file

```
controlplane ~ ➜ base64 -w 0 akshay.csr
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFWx6U05UY3lTT3A4bgp5RmpDLzQzVkRheWlJeGpBQnorRzM0akpyREJjWXoyTXo0OE42c3l3M01icnp1MDZRQndFMTBGblpZTU5GcTNtCkFSVm4zRTNrbXlFMHVGS0VUcWRqWmxVVExZOFhQNWVlanhtb2xScmJLOUI5eWkweDFjcjM5M2loNjFwSjNvRkEKLzEwaWNpUGtVcFhsSEIrNFVEY2hHTzkyZXBnMFo0UEZlKzg9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
controlplane ~ ➜  vi singReq.yml 

controlplane ~ ➜  k apply -f singReq.yml 
certificatesigningrequest.certificates.k8s.io/akshay created
```
https://github.com/mehediasif0001/Certified-Kubernetes-Administrator-CKA---Asif/blob/main/security/CertificateSigningRequest.yml
```
controlplane ~ ➜  k get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
akshay      5s    kubernetes.io/kube-apiserver-client           kubernetes-admin           24h                 Pending
csr-zfrtt   33m   kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued
```

task 3;
Approve the CSR Request

```
controlplane ~ ➜  k certificate approve akshay
certificatesigningrequest.certificates.k8s.io/akshay approved

controlplane ~ ➜  k get csr
NAME        AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
akshay      5m18s   kubernetes.io/kube-apiserver-client           kubernetes-admin           24h                 Approved,Issued
csr-zfrtt   38m     kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued
```

task 4;


```
controlplane ~ ➜  k get csr
NAME          AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
agent-smith   39s     kubernetes.io/kube-apiserver-client           agent-x                    <none>              Pending

controlplane ~ ➜  kubectl get csr agent-smith -o yaml
```

Reject that request.

```
controlplane ~ ➜  k certificate deny agent-smith
certificatesigningrequest.certificates.k8s.io/agent-smith denied

controlplane ~ ➜  k get csr
\NAME          AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
agent-smith   4m10s   kubernetes.io/kube-apiserver-client           agent-x                    <none>              Denied
```

 Delete the new CSR object

```
controlplane ~ ➜  k delete csr agent-smith
certificatesigningrequest.certificates.k8s.io "agent-smith" deleted
```
#network policy

How many network policies do you see in the environment? What is the name of the Network Policy?

```
controlplane ~ ➜  k get networkpolicies 
NAME             POD-SELECTOR   AGE
payroll-policy   name=payroll   2m34s
```

Which pod is the Network Policy applied on?

Use the command kubectl get netpol and look under the Pod Selector column.

```
controlplane ~ ➜  k describe networkpolicies payroll-policy
Name:         payroll-policy
Namespace:    default
Created on:   2026-02-15 21:19:37 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     name=payroll     <<<<---- this is pod label which is   Network Policy applied on
  Allowing ingress traffic:
    To Port: 8080/TCP
    From:
      PodSelector: name=internal  
  Not affecting egress traffic
  Policy Types: Ingress
```

```
controlplane ~ ➜  k get po --show-labels | grep name=payroll
payroll    1/1     Running   0          10m   name=payroll
```

◼ Task: Create Egress NetworkPolicy

• Create a NetworkPolicy named internal-policy in the default namespace
• Apply it to pods with label name=internal
• Allow egress traffic only to:
▸ mysql on TCP 3306
▸ payroll on TCP 8080
▸ DNS (kube-system) on TCP/UDP 53

• Block all other outbound traffic
l

ans : https://github.com/mehediasif0001/Certified-Kubernetes-Administrator-CKA---Asif/blob/main/security/NetworkPolicy.yml
