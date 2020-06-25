# Clean up

### Objectives
1. Ensure EKS Cluster and other AWS Services are removed

### Prereq

### Setup Steps

### 1. PVC and EBS volumes 

Verify there are no left over EBS volumes from the PVC's for test application

```bash
 aws ec2 describe-volumes --filters Name=tag:kubernetes.io/created-for/pv/name,Values=*     --query "Volumes[].{ID:VolumeId}"
```

Delete any ebs volumes found for the PVC's for the postgres test database 

### 2. EKS Clusters 

Verify we have not left any eks clusters running 

```bash
eksctl get clusters
```

To Delete it 
```bash
eksctl delete cluster --name $CLUSTER_NAME
```

### 3. Load Balancers

Verify there are no Load balancers running, ALB or otherwise

```bash
aws elbv2 describe-load-balancers --query "LoadBalancers[].LoadBalancerArn"
```

```bash
aws elb describe-load-balancers --query "LoadBalancerDescriptions[].DNSName"
```