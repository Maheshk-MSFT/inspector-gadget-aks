# inspector-gadget-aks
steps to inspector-gadget

Installing on Linux
===================
```
IG_VERSION=$(curl -s https://api.github.com/repos/inspektor-gadget/inspektor-gadget/releases/latest | jq -r .tag_name)
IG_ARCH=amd64
curl -sL https://github.com/inspektor-gadget/inspektor-gadget/releases/download/${IG_VERSION}/ig-linux-${IG_ARCH}-${IG_VERSION}.tar.gz | sudo tar -C /usr/local/bin -xzf - ig
ig version
```
---
```
sudo ig run trace_open:latest -c mycontainer
```
---
```
$ IG_VERSION=$(curl -s https://api.github.com/repos/inspektor-gadget/inspektor-gadget/releases/latest | jq -r .tag_name)
$ IG_ARCH=amd64
$ curl -sL https://github.com/inspektor-gadget/inspektor-gadget/releases/download/${IG_VERSION}/kubectl-gadget-linux-${IG_ARCH}-${IG_VERSION}.tar.gz  | sudo tar -C /usr/local/bin -xzf - kubectl-gadget
$ kubectl gadget version
```
---
```
kubectl gadget deploy
kubectl get pod -n gadget 
kubect gadget version
```
---
```
kubectl gadget run top_tcp:latest --podname store-front-bfb8f484c-dxt8j  --fields=k8s.containername,comm,pid,ppid,src,dst,sent,recieved

kubectl gadget run top_file:latest
kubectl gadget run top_tcp:latest
kubectl gadget top tcp
kubectl gadget top file![image](https://github.com/user-attachments/assets/862ec6c2-30d9-4d1d-9c47-3b234d50bcea)
```
