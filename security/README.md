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

