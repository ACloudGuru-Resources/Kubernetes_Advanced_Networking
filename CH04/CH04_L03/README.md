# Lab Name

### Objectives

1. Deploy Nodeport
2. Understand the drawbacks of using Nodeport Service Type

### Prereq

1. Running Kind cluster

### Setup Steps

1. Start kind cluster

```bash
kind create cluster --config ./kind.yml --name nodeport
```

```bash
± |master {1} S:5 U:4 ?:1 ✗| → kind create cluster --config kind.config --name nodeport
Creating cluster "nodeport" ...
 ✓ Ensuring node image (kindest/node:v1.16.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-nodeport"
You can now use your cluster with:

kubectl cluster-info --context kind-nodeport

Have a nice day! 👋
```

2. Check Node Labels

```bash
kubectl get nodes --show-labels
```

```bash
± |master {1} S:4 U:2 ?:3 ✗| → kubectl get nodes --show-labels
NAME                     STATUS   ROLES    AGE    VERSION   LABELS
nodeport-control-plane   Ready    master   2m1s   v1.16.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=nodeport-control-plane,kubernetes.io/os=linux,node-role.kubernetes.io/master=
nodeport-worker          Ready    <none>   88s    v1.16.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=nodeport-worker,kubernetes.io/os=linux
nodeport-worker2         Ready    <none>   87s    v1.16.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=nodeport-worker2,kubernetes.io/os=linux
nodeport-worker3         Ready    <none>   87s    v1.16.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=nodeport-worker3,kubernetes.io/os=linux
```

Get the IP address of node-worker

```bash
kubectl get nodes -o wide
```

```bash
NAME                     STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                  KERNEL-VERSION     CONTAINER-RUNTIME
nodeport-control-plane   Ready    master   36m   v1.16.3   172.17.0.5    <none>        Ubuntu Eoan Ermine (development branch)   4.19.76-linuxkit   containerd://1.3.0-27-g54658b88
nodeport-worker          Ready    <none>   35m   v1.16.3   172.17.0.3    <none>        Ubuntu Eoan Ermine (development branch)   4.19.76-linuxkit   containerd://1.3.0-27-g54658b88
nodeport-worker2         Ready    <none>   35m   v1.16.3   172.17.0.6    <none>        Ubuntu Eoan Ermine (development branch)   4.19.76-linuxkit   containerd://1.3.0-27-g54658b88
nodeport-worker3         Ready    <none>   35m   v1.16.3   172.17.0.4    <none>        Ubuntu Eoan Ermine (development branch)   4.19.76-linuxkit   containerd://1.3.0-27-g54658b88
```

3. Deploy Service

kubectl apply -f nodeport-service.yml

4. Let's Investigate

4.1 Deploy our trusted DNS utils pod

```bash
kubectl apply -f dnsutils.yml
```

4.2 Get the IP address of any node other than the one we labeled

```bash
kubectl get nodes -o wide
```

```bash
NAME                     STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                  KERNEL-VERSION     CONTAINER-RUNTIME
nodeport-control-plane   Ready    master   36m   v1.16.3   172.17.0.5    <none>        Ubuntu Eoan Ermine (development branch)   4.19.76-linuxkit   containerd://1.3.0-27-g54658b88
nodeport-worker          Ready    <none>   35m   v1.16.3   172.17.0.3    <none>        Ubuntu Eoan Ermine (development branch)   4.19.76-linuxkit   containerd://1.3.0-27-g54658b88
nodeport-worker2         Ready    <none>   35m   v1.16.3   172.17.0.6    <none>        Ubuntu Eoan Ermine (development branch)   4.19.76-linuxkit   containerd://1.3.0-27-g54658b88
nodeport-worker3         Ready    <none>   35m   v1.16.3   172.17.0.4    <none>        Ubuntu Eoan Ermine (development branch)   4.19.76-linuxkit   containerd://1.3.0-27-g54658b88
```

4.3 Test Connectivity

External Communcation uses the nodeport

```bash
kubectl exec -it dnsutils -- wget -q -O- 172.17.0.3:30040/host
{"message":"NODE: nodeport-worker, POD IP:10.244.1.9"}

kubectl exec -it dnsutils -- wget -q -O-  172.17.0.4:30040/host
{"message":"NODE: nodeport-worker, POD IP:10.244.1.9"}

kubectl exec -it dnsutils -- wget -q -O-  172.17.0.6:30040/host
{"message":"NODE: nodeport-worker, POD IP:10.244.1.9"}
```

Internal Cluster Communication uses the port

```bash
kubectl exec -it dnsutils -- wget -q -O- 10.244.1.9:8080/host
{"message":"NODE: nodeport-worker, POD IP:10.244.1.9"}

kubectl exec -it dnsutils -- wget -q -O- nodeport-service.default.svc.cluster.local:8080/host
{"message":"NODE: nodeport-worker, POD IP:10.244.1.9"}
```

4.4 Review IP tables rules

There is an IPtables rule on

```bash
docker exec -it nodeport-worker3 iptables -L -t nat | grep nodeport-service

KUBE-MARK-MASQ  tcp  --  anywhere             anywhere             /* default/nodeport-service:echo */ tcp dpt:30040
KUBE-SVC-SR75RJIBNEHN2H65  tcp  --  anywhere             anywhere             /* default/nodeport-service:echo */ tcp dpt:30040
KUBE-MARK-MASQ  tcp  -- !10.244.0.0/16        10.100.118.139       /* default/nodeport-service:echo cluster IP */ tcp dpt:8080
KUBE-SVC-SR75RJIBNEHN2H65  tcp  --  anywhere             10.100.118.139       /* default/nodeport-service:echo cluster IP */ tcp dpt:8080
```

```bash
docker exec -it nodeport-worker2 iptables -L -t nat | grep nodeport-service

KUBE-MARK-MASQ  tcp  --  anywhere             anywhere             /* default/nodeport-service:echo */ tcp dpt:30040
KUBE-SVC-SR75RJIBNEHN2H65  tcp  --  anywhere             anywhere             /* default/nodeport-service:echo */ tcp dpt:30040
KUBE-MARK-MASQ  tcp  -- !10.244.0.0/16        10.100.118.139       /* default/nodeport-service:echo cluster IP */ tcp dpt:8080
KUBE-SVC-SR75RJIBNEHN2H65  tcp  --  anywhere             10.100.118.139       /* default/nodeport-service:echo cluster IP */ tcp dpt:8080
```

```bash
docker exec -it nodeport-worker iptables -L -t nat | grep nodeport-service

KUBE-MARK-MASQ  tcp  --  anywhere             anywhere             /* default/nodeport-service:echo */ tcp dpt:30040
KUBE-SVC-SR75RJIBNEHN2H65  tcp  --  anywhere             anywhere             /* default/nodeport-service:echo */ tcp dpt:30040
KUBE-MARK-MASQ  tcp  -- !10.244.0.0/16        10.100.118.139       /* default/nodeport-service:echo cluster IP */ tcp dpt:8080
KUBE-SVC-SR75RJIBNEHN2H65  tcp  --  anywhere             10.100.118.139       /* default/nodeport-service:echo cluster IP */ tcp dpt:8080
```

If we follow the chain KUBE-SVC-SR75RJIBNEHN2H65

You can see that it Destination NAT to the pod ip 10.244.1.9

```bash
docker exec -it nodeport-worker3 iptables -L KUBE-SVC-SR75RJIBNEHN2H65 -t nat
Chain KUBE-SVC-SR75RJIBNEHN2H65 (2 references)
target     prot opt source               destination
KUBE-SEP-ZGBSGCBJNGMY3BBA  all  --  anywhere             anywhere

docker exec -it nodeport-worker3 iptables -L KUBE-SEP-ZGBSGCBJNGMY3BBA -t nat
Chain KUBE-SEP-ZGBSGCBJNGMY3BBA (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  all  --  10.244.1.9           anywhere
DNAT       tcp  --  anywhere             anywhere             tcp to:10.244.1.9:8080
```