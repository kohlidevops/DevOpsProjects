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





