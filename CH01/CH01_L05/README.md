# Tools Needed for A Cloud Guru Advance Network on Kubernetes in AWS Labs  

### Objectives

Install the list of tools below

1. VirtualBox
2. Docker
3. kubectl
4. KIND (Kubernetes in Docker)
5. aws cli
6. eksctl

## [VirtualBox](https://www.virtualbox.org/)

VirtualBox is a general-purpose full virtualization for x86 hardware.

#### 1. PreReq

Check to make sure your laptop supports virtualization

Link to supported platforms

https://www.virtualbox.org/manual/UserManual.html#hostossupport

CLI example on Mac

```bash
sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
```

#### 2. Download

[Download](https://www.virtualbox.org/wiki/Downloads) and install Virtualbox for your Operating System

#### 3. Install

[Installation details](https://www.virtualbox.org/manual/UserManual.html#installation)

#### 4. Verify

Download and run a Virtualbox image

There are many options when working with Virtualbox

- [Vagrant](https://app.vagrantup.com/boxes/search)
- [OS Boxes](https://www.osboxes.org/virtualbox-images/)
- Manual - Download a copy of an OS and [run through creating your first VM](https://www.virtualbox.org/manual/UserManual.html#intro-running)

## [Docker](https://docs.docker.com/install/)

Docker is an open source container runtime

#### 1. PreReq

- [Docker Hub account](https://hub.docker.com/sso/start)
- See the install for system requirements for your operating system.

#### 2. Download
Download Docker for your operating system

- [Mac](https://docs.docker.com/docker-for-mac/)
- [Windows](https://docs.docker.com/docker-for-windows/)

#### 3. Install

- [Mac](https://docs.docker.com/docker-for-mac/install/)
- [Windows](https://docs.docker.com/docker-for-windows/install/)

#### 4. Verify

`docker --version`

`docker run hello-world`

##	[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)

kubectl is the main entry point into the kubernetes API.  

#### 1. PreReq

You must use a kubectl version that is within one minor version difference of your cluster. 
For example, a v1.2 client should work with v1.1, v1.2, and v1.3 master. Using the latest version of 
kubectl helps avoid unforeseen issues.

#### 2. Download

See Install

#### 3. Install

- [Mac](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos)
- [Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)
- [Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos)

#### 4. Verify

`kubectl version`

##	[KIND (Kubernetes in Docker)](https://kind.sigs.k8s.io/)

#### 1. PreReq

- Docker

#### 2. Download

https://kind.sigs.k8s.io/docs/user/quick-start

#### 3. Install

https://kind.sigs.k8s.io/docs/user/quick-start

#### 4. Verify

`kind create cluster`

## [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)

AWS CLI is one of the ways to programmatically access your AWS resources. 

#### 1. PreReq

- Python

#### 2. Download

See Install

#### 3. Install

- [Mac/Linux](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux-mac.html)
- [Windows](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html)

#### 4. Verify

`aws --version`

`aws-cli/1.16.290 Python/3.7.5 Darwin/19.2.0 botocore/1.13.26`

##	[eksctl](https://eksctl.io/)

eksctl is a CLI tool created by weave to help manage kubernetes clusters on AWS easily.

#### 1. PreReq

- AWS CLI

#### 2. Download

See Install

#### 3. Install

https://eksctl.io/introduction/installation/

#### 4. Verify

`eksctl version`

`eksctl create cluster`

A cluster will be created with default parameters

- exciting auto-generated name, e.g. “fabulous-mushroom-1527688624”
- 2x m5.large nodes (this instance type suits most common use-cases, and is good value for money)
- use official AWS EKS AMI
- us-west-2 region
- dedicated VPC (check your quotas)
- using static AMI resolver

Lets go ahead and delete that cluster

`eksctl delete cluster --name CLUSTERNAME`

### [Network Troubleshooting image](https://github.com/strongjz/netshoot?organization=strongjz&organization=strongjz)

Troubleshooting applications pods can be difficult from outside the cluster, running a container with troubleshooting tools
inside the cluster can be helpful.

#### 1. PreReq

- Docker

#### 2. Download

Build it yourself, Dockerfile is located in this [repo here](https://github.com/strongjz/netshoot?organization=strongjz&organization=strongjz)

OR

`docker pull acloudgurulabs/course_kubernetes_advanced_networking:latest`

#### 3. Install

```bash
kubectl run --generator=run-pod/v1 tmp-shell --rm -i --tty --image acloudgurulabs/course_kubernetes_advanced_networking:latest -- /bin/bash
```

#### 4. Verify

The Kubectl run should drop you into a shell

```bash
ping google.com
```
