What is the Common Name (CN) configured on the Kube API Server Certificate?


Run the command 
```
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
```
and look for Subject CN.

task2:
 
