# Overlay Network Lab

### Objectives
1. Deploy EKS Cluster
2. Deploy Cilium CNI 
3. Verify deployment

### Prereq
1. [Eksctl installed](CH01/CH01_L05)
2. [AWS IAM authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)  
3. [Helm installed](https://helm.sh/docs/intro/install/)
4. [AWS Account with credentials for CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

### Setup Steps
1. Deploy Cluster
2. Disable the aws-node DaemonSet
3. Deploy Cilium 
4. Validate Install

### 1. Deploy Cluster

```bash
eksctl create cluster -n test-cluster -N 0
```

### 2. Disable the aws-node DaemonSet


```bash
kubectl -n kube-system set image daemonset/aws-node aws-node=docker.io/spaster/alpine-sleep
```


### 3. Deploy Cilium 

3.1 Add helm repo to Cilium 

```bash
helm repo add cilium https://helm.cilium.io/
```

3.2 Deploy Cilium

```bash
helm install cilium cilium/cilium --version 1.7.1 \
  --namespace kube-system \
  --set global.eni=true \
  --set global.egressMasqueradeInterfaces=eth0 \
  --set global.tunnel=disabled \
  --set global.nodeinit.enabled=true
```

3.3 Scale up the cluster 

Get the node group for our cluster

```bash
eksctl get nodegroup --cluster overlay-cluster```

Scale up that node group 

```bash
eksctl scale nodegroup --cluster overlay-cluster -n ng-xxxxxxxx-N 3
```

### 4. Validate Install

4.1 Wait for all components to come 

```bash
kubectl -n kube-system get pods --watch
``` 
4.2 Connectivity Test 

```bash 
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/v1.7/examples/kubernetes/connectivity-check/connectivity-check.yaml
```


