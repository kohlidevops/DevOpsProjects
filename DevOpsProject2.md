# To deploying Application through DevOps CI/CD pipelines using Git, Jenkins, Trivy, KubeAudit, Maven, Nexus, SonarQube, Docker & Kubernetes and Monitoring using Promotheus, Grafana, Blackbox Exporter & Node Exporter


  ![Untitled Diagram11](https://github.com/user-attachments/assets/278ad3de-1a8c-4af1-8f35-d2d94983172c)


- Set up a Kubernetes cluster in an AWS environment

- Security scan by KubeAudit on Kubernetes cluster

- Set up VM's for Jenkins,Nexus,Maven,and SonarQube tools

- Git Bash and GitHub

- Customize the Jenkins

- Create a Jenkins pipeline job to check out the project

- Compile and run unit test cases on source code

- Trivy tool - Vulnerability Scan on Source Code

- SonarQube - Code quality tool for better code

- Build the package: Using maven tool

- Upload the artifact to the Nexus Repository

- Build and Tag the Docker Image

- Docker Image Scanning by Trivy tool

- Push the docker image to DockerHub

- Deploy the application to a Kubernetes cluster environment

- Monitoring with Prometheus and Grafana

## To Set up a Kubernetes cluster in an AWS environment

Launch 3 EC2 ubuntu-22 instances - one for Master another 2 for Worker nodes

<img width="952" alt="image" src="https://github.com/user-attachments/assets/9a91e755-d33a-4278-b39c-ee0dbe06787a" />

- To SSH to master node and connect other worker nodes from master using private key

- To perform below commands in master and worker nodes

master node

```
sudo swapoff -a
sudo hostnamectl set-hostname master
sudo su
su - ubuntu
```

worker node 1

```
sudo swapoff -a
sudo hostnamectl set-hostname worker1
sudo su
su - ubuntu
```

worker node 2

```
sudo swapoff -a
sudo hostnamectl set-hostname worker2
sudo su
su - ubuntu
```

- To SSh all nodes and update all the Public IP with their hostname

```
sudo vi/etc/hosts

3.111.42.75 master
13.126.22.62 worker1
13.127.133.146 worker2
```

Now you can SSH from one node to another node using their public ip

- To install containerd on all nodes using below script

```
sudo vi containerd-install.sh

https://github.com/kohlidevops/DevOpsProjects/blob/main/containerd-install.sh

sudo chmod u+x containerd-install.sh
sh containerd-install.sh
service containerd status
```

- To install kubeadm, kubelet and kubectl on all nodes using below script

```
sudo vi k8s-install.sh

https://github.com/kohlidevops/DevOpsProjects/blob/main/k8s-install.sh

sudo chmod u+x k8s-install.sh
sh k8s-install.sh
kubeadm version
kubectl version
service kubelet status //in-active - because not initialized the cluster
```

- To initialize the kubeadm in master node

```
sudo kubeadm init

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes // not ready
kubectl get pod -A //The connection to the server 172.31.10.77:6443 was refused - did you specify the right host or port? - For this, you do follow the below things
journalctl -u kubelet.service //so many errors - by default cgroups using to manager the container - but it should use systemd - For this
sudo vi /etc/containerd/config.toml // To find and change to true "SystemdCgroup = true"
sudo service containerd restart
sudo service kubelet restart
kubectl aaply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
kubectl get pods -A
kubectl get nodes
```

<img width="787" alt="image" src="https://github.com/user-attachments/assets/c19c4086-bc9a-4717-9baa-746923232726" />

- To join worker nodes to kubernetes cluster

First run below command to get the token in master node

```
kubeadm token create --print-join-command
//sudo kubeadm join 172.31.10.77:6443 --token 8lcuoq.bwefohhp5e2opkf4 --discovery-token-ca-cert-hash sha256:4e45c2a979adb435ac3484eeee072b08689b38e849e9478a0cbd5c4a95b233a7
```

Now run below command in worker nodes

```
sudo kubeadm join 172.31.10.77:6443 --token 8lcuoq.bwefohhp5e2opkf4 --discovery-token-ca-cert-hash sha256:4e45c2a979adb435ac3484eeee072b08689b38e849e9478a0cbd5c4a95b233a7
```

Now check with master nodes

```
kubectl get nodes
```

<img width="554" alt="image" src="https://github.com/user-attachments/assets/fbfc4bf3-da9a-49b1-9eea-d1bb5d0e3985" />


## To Security Scan by KubeAudit on Kubernetes Cluster

- kubeaudit, an open source tool created by the folks at Shopify, can be used to perform a security audit of Kubernetes clusters to find common low hanging fruits that are often exploited by attackers.

- kubeaudit can be run in 3 modes based on the location of the cluster and your access.

1. Manifest mode - Use this mode to audit Kubernetes YAML manifests before applying them to a cluster. This checks the security best practices in deployment.yaml before deploying it.

2. Local mode - Use this mode when you have kubectl access to a cluster but want to audit local configurations.

3. Cluster mode - Use this mode to audit live resources in a running cluster.


- To download and install KubeAudit

Refer - https://github.com/Shopify/kubeaudit/releases

SSH to master node

```
sudo mkdir kubeaudit
cd kubeaudit/
sudo wget https://github.com/Shopify/kubeaudit/releases/download/v0.22.1/kubeaudit_0.22.1_linux_amd64.tar.gz
sudo tar -xvzf kubeaudit_0.22.1_linux_amd64.tar.gz 
ls -lh
sudo mv kubeaudit /usr/local/bin/
```

- KubeAudit to Scan the Manifest file

SSH to master node

```
cd /home/ubuntu/kubeaudit
sudo nano pod.yaml

https://github.com/kohlidevops/DevOpsProjects/blob/main/pod.yaml

kubeaudit all -f pod.yaml //There are lot of errors
sudo kubeaudit autofix -f pod.yaml -o fixed-pod.yaml  //To reduce the errors and warnings
kubeaudit all -f fixed-pod.yaml //Now check
```






