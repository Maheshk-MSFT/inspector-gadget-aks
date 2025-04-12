# What is Inspektor Gadget (AKS IG) ?
-------------------------------------
- tools for debugging and inspecting Linux and Kubernetes (AKS too)
- tools ("gadgets") are implemented as eBPF programs
- used to gather low-level kernel data to provide insights into specific system scenarios
- seamless connection exists between low-level insights and their corresponding high-level context - helps troubleshooting process and the collection of relevant information.

using IG, we cna trace following events in our AKS cluster, 
a. Process creation
b. File access
c. Network activity, such as TCP connections or DNS resolution

---
1. Disk-intensive applications - High memory or CPU usage, or inconsistent node readiness - use top_blockio
2. "It's always DNS" - High application latency, time-outs, or poor end-user experience - use Trace_dns
3. File system access	Application misbehaves or can't function correctly - use Trace_open
4. Remote code execution (RCE) - Unauthorized code execution such as cryptojacking that's evident in high CPU usage during application idle periods - use Trace_exec
---
# inspector-gadget-aks
Steps to install inspector-gadget at our AKS Cluster 

Installing on Linux (WSL - Ubuntu20)
-------------------------------------
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

How to monitor the network/data IO between the PODS

<img width="955" alt="11" src="https://github.com/user-attachments/assets/1349761d-0fb4-4ecf-92e3-59a61b160b09" />
---
<img width="959" alt="22" src="https://github.com/user-attachments/assets/c4a179a6-584d-4e93-aba1-89f47a62359d" />
---
Additonal reference:  </br>
1. https://inspektor-gadget.io/docs/latest/gadgets/top_blockio </br>
2. https://inspektor-gadget.io/docs/latest/gadgets/top_file </br>
3. https://inspektor-gadget.io/docs/latest/gadgets/top_tcp  </br>
4. https://learn.microsoft.com/en-us/troubleshoot/azure/azure-kubernetes/logs/capture-system-insights-from-aks#what-is-inspektor-gadget </br>

---
# Single click installation script for AKS 

```
# Create a resource group
export RANDOM_ID="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="myResourceGroup$RANDOM_ID"
export REGION="eastus"
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION

# Create AKS Cluster
export MY_AKS_CLUSTER_NAME="myAKSCluster$RANDOM_ID"
az aks create \
  --resource-group $MY_RESOURCE_GROUP_NAME \
  --name $MY_AKS_CLUSTER_NAME \
  --location $REGION \
  --no-ssh-key

# Install kubectl
if ! [ -x "$(command -v kubectl)" ]; then az aks install-cli; fi

# Configure credentials
az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AKS_CLUSTER_NAME --overwrite-existing

# Veryify conncetion
kubectl get nodes

# Installing the kubectl plugin: `gadget`
IG_VERSION=$(curl -s https://api.github.com/repos/inspektor-gadget/inspektor-gadget/releases/latest | jq -r .tag_name)
IG_ARCH=amd64
mkdir -p $HOME/.local/bin
export PATH=$PATH:$HOME/.local/bin
curl -sL https://github.com/inspektor-gadget/inspektor-gadget/releases/download/${IG_VERSION}/kubectl-gadget-linux-${IG_ARCH}-${IG_VERSION}.tar.gz  | tar -C $HOME/.local/bin -xzf - kubectl-gadget

kubectl gadget version

# Installing Inspektor Gadget in the cluster
kubectl gadget deploy
kubectl gadget version
```


