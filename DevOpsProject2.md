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

- KubeAudit to Scan the Cluster

SSh to master node

```
kubeaudit all
```

If you are not installing KubeAudit on the cluste, but you want to audit the cluster - There is a way to run a kubeaudit container using kubectl command

```
kubectl run kubeaudit --image shopify/kubeaudit --rm -it //All checks are completed
```

- KubeAudit to Scan the Local

SSH to master node

```
kubectl config current-context //To get the current context
kubeaudit all --kubeconfig <path-to-config> --context <my-cluster-name>
kubeaudit all --kubeconfig /home/ubuntu/.kube/config --context kubernetes-admin@kubernetes
```


## To Setup Jenkins, Maven, SonarQube and Nexus

- Create a VM for SonarQube and Nexus

To launch 2 EC2 ubuntu (22.04) instances with T3.medium


<img width="785" alt="image" src="https://github.com/user-attachments/assets/de8c958e-0e87-4396-ba9c-6ea872e1db47" />


- Install Docker in SonarQube Server

SSH to SonarQube Server

Refer - https://docs.docker.com/engine/install/ubuntu/

```
sudo hostnamectl set-hostname Sonarqube
sudo su
su - ubuntu
sudo apt-get update -y
sudo apt-get update

sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo chmod 666 /var/run/docker.sock
docker run hello-world
docker image ls
```

- Install SonarQube using Docker

```
docker login -u latchudevops
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
docker ps
```

Now you can access the sonarqube portal with server-ip:9000

```
default username - admin
default password - admin
```

Now update the custom password

<img width="896" alt="image" src="https://github.com/user-attachments/assets/78e5e760-045e-431e-a6c2-1cf8f684b304" />


- Install Docker in Nexus Server

SSH to Nexus Server

Refer - https://docs.docker.com/engine/install/ubuntu/

```
sudo hostnamectl set-hostname Nexus
sudo su
su - ubuntu
sudo apt-get update -y

sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo chmod 666 /var/run/docker.sock
docker run hello-world
docker image ls
```

- Install Nexus using Docker

```
docker login -u latchudevops
docker run -d --name Nexus -p 8081:8081 sonatype/nexus3
docker ps
```

- To login to the Nexus container

```
docker exec -it <container-id> /bin/bash
docker exec -it 02e776cc5c95 /bin/bash
$cd sonatype-work/nexus3/
$cat admin.password
```

You can access the Nexus by ServerIP:8081

<img width="944" alt="image" src="https://github.com/user-attachments/assets/bd4de10d-f521-411b-a1e2-82c7733c2937" />

```
default username - admin
default password -
//To change the password and enable anonymous access to start
```

<img width="941" alt="image" src="https://github.com/user-attachments/assets/fb1802ab-c043-4e2c-be6d-fc67139c1b87" />

- Install Jenkins, Docker and Configure

Launch one Ubuntu-22 EC2 instance with T3.medium and install jenkins on it

<img width="780" alt="image" src="https://github.com/user-attachments/assets/10da54aa-d025-424a-ada4-d43806d80ead" />

SSH to server

Refer - https://www.jenkins.io/doc/book/installing/linux/

```
sudo hostnamectl set-hostname Jenkins
sudo su
su - ubuntu
sudo apt-get update -y
sudo apt install openjdk-17-jre-headless
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y

sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo chmod 666 /var/run/docker.sock
docker run hello-world
docker image ls
```

To access the jenkins, install suggested plugins and create a new admin user

<img width="948" alt="image" src="https://github.com/user-attachments/assets/710130b4-41bb-48ea-9f0a-4be20cf3a920" />


## To Setup Github repo

To create a repo named techgame as a private in Github

<img width="791" alt="image" src="https://github.com/user-attachments/assets/a537747c-4949-40d1-a92a-fa825e9c3e22" />

To create a Github token and keep it safely to use

Github > Settings > Developer setting > Generate classic token

<img width="863" alt="image" src="https://github.com/user-attachments/assets/87030a64-507f-4004-b582-3d8d6ec25008" />

Download the source code in local machine using below link

```
https://github.com/kohlidevops/DevOpsProjects/blob/main/SourceCode.zip
```

To clone the repo in local machine and open gitbash in below path

C:\Users\latchu\Downloads\SourceCode\SourceCode\techgame

```
//Gitbash open and clone
https://github.com/kohlidevops/techgame.git
//Place all the files inside this folder
```









