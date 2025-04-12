# What is Inspektor Gadget?
---------------------------
Inspektor Gadget is a framework that's designed for building, packaging, deploying, and running tools that are dedicated to debugging and inspecting Linux and Kubernetes systems. These tools ("gadgets") are implemented as eBPF programs. Their primary goal is to gather low-level kernel data to provide insights into specific system scenarios. The Inspektor Gadget framework manages the association of the collected data by using high-level references, such as Kubernetes resources. This integration makes sure that a seamless connection exists between low-level insights and their corresponding high-level context. The integration streamlines the troubleshooting process and the collection of relevant information.

using IG, we cna trace following events in our AKS cluster, 
a. Process creation
b. File access
c. Network activity, such as TCP connections or DNS resolution

Disk-intensive applications - High memory or CPU usage, or inconsistent node readiness - use top_blockio
"It's always DNS" - High application latency, time-outs, or poor end-user experience - use Trace_dns
File system access	Application misbehaves or can't function correctly - use Trace_open
Remote code execution (RCE) - Unauthorized code execution such as cryptojacking that's evident in high CPU usage during application idle periods - use Trace_exec

----

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
How to monitor the network/data IO between the PODS

<img width="955" alt="11" src="https://github.com/user-attachments/assets/1349761d-0fb4-4ecf-92e3-59a61b160b09" />

<img width="959" alt="22" src="https://github.com/user-attachments/assets/c4a179a6-584d-4e93-aba1-89f47a62359d" />

----
Additonal reference: 

https://inspektor-gadget.io/docs/latest/gadgets/top_blockio
https://inspektor-gadget.io/docs/latest/gadgets/top_file
https://inspektor-gadget.io/docs/latest/gadgets/top_tcp
https://learn.microsoft.com/en-us/troubleshoot/azure/azure-kubernetes/logs/capture-system-insights-from-aks#what-is-inspektor-gadget
-----------------------
Script for AKS 

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


