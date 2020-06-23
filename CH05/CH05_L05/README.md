# AWS App Mesh Lab

### Objectives
1. Deploy EKS Cluster
2. Deploy AWS App Mesh Components
3. Deploy Test Application 
4. Verify
5. Clean up

### Prereq

### Setup Steps

1. Deploy EKS Cluster
```bash
eksctl create cluster -f eks-config.yml
```

```bash
[ℹ]  eksctl version 0.20.0
[ℹ]  using region us-west-2
[ℹ]  setting availability zones to [us-west-2b us-west-2c us-west-2d]
[ℹ]  subnets for us-west-2b - public:192.168.0.0/19 private:192.168.96.0/19
[ℹ]  subnets for us-west-2c - public:192.168.32.0/19 private:192.168.128.0/19
[ℹ]  subnets for us-west-2d - public:192.168.64.0/19 private:192.168.160.0/19
[ℹ]  nodegroup "app-mesh-lab" will use "ami-06e2c973f2d0373fa" [AmazonLinux2/1.16]
[ℹ]  using Kubernetes version 1.16
[ℹ]  creating EKS cluster "app-mesh-lab" in "us-west-2" region with un-managed nodes
[ℹ]  1 nodegroup (app-mesh-lab) was included (based on the include/exclude rules)
[ℹ]  will create a CloudFormation stack for cluster itself and 1 nodegroup stack(s)
[ℹ]  will create a CloudFormation stack for cluster itself and 0 managed nodegroup stack(s)
[ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-west-2 --cluster=app-mesh-lab'
[ℹ]  CloudWatch logging will not be enabled for cluster "app-mesh-lab" in "us-west-2"
[ℹ]  you can enable it with 'eksctl utils update-cluster-logging --region=us-west-2 --cluster=app-mesh-lab'
[ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "app-mesh-lab" in "us-west-2"
[ℹ]  2 sequential tasks: { create cluster control plane "app-mesh-lab", 2 parallel sub-tasks: { no tasks, create nodegroup "app-mesh-lab" } }
[ℹ]  building cluster stack "eksctl-app-mesh-lab-cluster"
[ℹ]  deploying stack "eksctl-app-mesh-lab-cluster"
[ℹ]  building nodegroup stack "eksctl-app-mesh-lab-nodegroup-app-mesh-lab"
[ℹ]  --nodes-min=3 was set automatically for nodegroup app-mesh-lab
[ℹ]  --nodes-max=3 was set automatically for nodegroup app-mesh-lab
[ℹ]  deploying stack "eksctl-app-mesh-lab-nodegroup-app-mesh-lab"
[ℹ]  waiting for the control plane availability...
[✔]  saved kubeconfig as "/Users/strongjz/.kube/config"
[ℹ]  no tasks
[✔]  all EKS cluster resources for "app-mesh-lab" have been created
[ℹ]  adding identity "arn:aws:iam::725406136353:role/eksctl-app-mesh-lab-nodegroup-app-NodeInstanceRole-JRDBNFW2PQQM" to auth ConfigMap
[ℹ]  nodegroup "app-mesh-lab" has 0 node(s)
[ℹ]  waiting for at least 3 node(s) to become ready in "app-mesh-lab"
[ℹ]  nodegroup "app-mesh-lab" has 3 node(s)
[ℹ]  node "ip-192-168-16-115.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-56-247.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-91-149.us-west-2.compute.internal" is ready
[ℹ]  kubectl command should work with "/Users/strongjz/.kube/config", try 'kubectl get nodes'
[✔]  EKS cluster "app-mesh-lab" in "us-west-2" region is ready
```


2. Deploy AWS App Mesh Components

AWS has released v1.0.0 of the AWS App Mesh controller 

This requires Helm Version, if you do not have helm v3 installed [follow instructions here](CH01/CH01_L05/helm_install.md)

```bash
helm repo add eks https://aws.github.io/eks-charts
```
```bash
"eks" has been added to your repositories
```
Verify 
```bash 
helm repo list | grep eks-charts
```
```bash
eks   	https://aws.github.io/eks-charts
```


2.1 IAM Service Account

2.1.1 Create a namespace for the app mesh controller

```bash
kubectl apply -f app-mesh-ns.yml
```
```bash
namespace/app-mesh-lab created
```

2.1.2 Create your OIDC identity provider for the cluster
```bash
eksctl utils associate-iam-oidc-provider \
  --cluster app-mesh-lab \
  --approve
```
```bash
[ℹ]  eksctl version 0.20.0
[ℹ]  using region us-west-2
[ℹ]  will create IAM Open ID Connect provider for cluster "app-mesh-lab" in "us-west-2"
[✔]  created IAM Open ID Connect provider for cluster "app-mesh-lab" in "us-west-2"
```

2.1.3 Create an IAM role for the appmesh controller service account
```bash 
eksctl create iamserviceaccount \
  --cluster app-mesh-lab \
  --namespace app-mesh-lab \
  --name app-mesh-lab-controller \
  --attach-policy-arn  arn:aws:iam::aws:policy/AWSCloudMapFullAccess,arn:aws:iam::aws:policy/AWSAppMeshFullAccess \
  --override-existing-serviceaccounts \
  --approve
```
Output
```bash
[ℹ]  eksctl version 0.20.0
[ℹ]  using region us-west-2
[ℹ]  1 iamserviceaccount (app-mesh-lab/appmesh-controller) was included (based on the include/exclude rules)
[!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
[ℹ]  1 task: { 2 sequential sub-tasks: { create IAM role for serviceaccount "app-mesh-lab/appmesh-controller", create serviceaccount "app-mesh-lab/appmesh-controller" } }
[ℹ]  building iamserviceaccount stack "eksctl-app-mesh-lab-addon-iamserviceaccount-app-mesh-lab-appmesh-controller"
[ℹ]  deploying stack "eksctl-app-mesh-lab-addon-iamserviceaccount-app-mesh-lab-appmesh-controller"
[ℹ]  created serviceaccount "app-mesh-lab/appmesh-controller"
```

2.2 App Mesh Controller 

2.2.1 Install App Mesh Controller 

 ```bash
export AWS_REGION=us-west-2

helm install appmesh-controller eks/appmesh-controller \
  --namespace app-mesh-lab \
  --set region=${AWS_REGION} \
  --set serviceAccount.create=false \
  --set serviceAccount.name=app-mesh-lab-controller
```
Output
```bash
NAME: appmesh-controller
LAST DEPLOYED: Mon Jun 22 20:26:56 2020
NAMESPACE: app-mesh-lab
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
AWS App Mesh controller installed!
```
Verify
```bash
kubectl -n app-mesh-lab get all
```
Output
```bash
NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/appmesh-controller-webhook-service   ClusterIP   10.100.99.120   <none>        443/TCP   55s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/appmesh-controller   0/1     0            0           55s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/appmesh-controller-96445c45c   1         0         0       55s

```

2.3 App mesh components for Application 

2.3.1 Deploy the Mesh

```bash
kubectl apply -f app-mesh.yml
```
Output
```bash
{
    "mesh": {
        "meshName": "app-mesh-lab",
        "metadata": {
            "arn": "arn:aws:appmesh:us-west-2::mesh/app-mesh-lab",
            "createdAt": "2020-06-22T21:05:18.332000-04:00",
            "lastUpdatedAt": "2020-06-22T21:05:18.332000-04:00",
            "uid": "f80b4ea8-ff40-46ed-bf63-aed217fe986c",
            "version": 1
        },
        "spec": {},
        "status": {
            "status": "ACTIVE"
        }
    }
}

```

Verify the App Mesh is active
```bash
aws appmesh describe-mesh --mesh-name app-mesh-lab
```

2.3.2 Now will deploy the Virtual Service, Routers and Nodes for the Ping, Data and Admin Applications
```bash
kubectl apply -f app-mesh-virtual.yml
```
Output
```bash
```

Verify the Virtual Service in AWS
aws appmesh list-virtual-services --mesh-name app-mesh-lab

aws appmesh list-virtual-nodes --mesh-name app-mesh-lab

aws appmesh list-virtual-routers --mesh-name app-mesh-lab


3. Deploy Test Application 

3.1 DNS Utils
```bash
kubectl apply -f dnsutils.yml
```
Output
```bash
pod/dnsutils created

kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
dnsutils   1/1     Running   0          12s
```

3.2 Database Deployment
```bash
kubectl apply -f database.yml
```
Output
```bash
service/postgres created
configmap/postgres-config created
statefulset.apps/postgres created
```

3.3 Application Deployment
```bash
kubectl apply -f app.yml
```
Output
```bash
deployment.apps/ping created
deployment.apps/data created
deployment.apps/admin created
```

4. Verify App Mesh Connectivity 




5. Clean up 

Delete the IAM account
```bash
 eksctl delete iamserviceaccount    --cluster app-mesh-lab   --namespace app-mesh-lab   --name app-mesh-lab-controller
 ```

Delete the Mesh 
```bash
kubectl delete -f app-mesh.yml
```

Delete the Cluster
```bash
 eksctl delete cluster -f eks-config.yml
```
 



