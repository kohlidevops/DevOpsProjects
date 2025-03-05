# Continuous Integration and Continuous Deployment on Self-hosted Kubernetes Cluster

# To deploying Application through DevOps CI/CD pipelines using Git, Jenkins, Trivy, KubeAudit, Maven, Nexus, SonarQube, Docker & Kubernetes and Monitoring using Promotheus, Grafana, Blackbox Exporter & Node Exporter




  ![RealTimeDevOps](https://github.com/user-attachments/assets/519c9d8c-6d6a-41fe-8f49-bad32b1b5a4e)





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
cd techgame
//Gitbash open and clone
https://github.com/kohlidevops/techgame.git
//Place all the files inside this folder
ls -lh
git add .
git commit -am "source code added"
git push
```


<img width="926" alt="image" src="https://github.com/user-attachments/assets/2946facc-cc5e-402f-ad0c-a1cda78167b7" />

Once push the code the repo then you can able to see the updated code in Github


## To Customize the Jenkins

- Install the required Plugins in Jenkins console

Login to the Jenkins console > Manage Jenkins > Plugins > install below plugins

```
Eclipse Temurin installer
Maven Integration
Config File Provider
Pipeline Maven Integration
SonarQube Scanner
Docker
Docker Pipeline
Kubernetes
Kubernetes Client API
Kubernetes Credentials
Kubernetes CLI
```

- To Configure tools - JDK, SonarQube Scanner, Maven and Docker

Manage Jenkins > tools > JDK installation > Add JDK

```
Name - jdk17
Choose - Install automatically
Install from adoptium.net - Version - jdk-17+35
```

<img width="792" alt="image" src="https://github.com/user-attachments/assets/00d60375-aa98-4740-adf9-337c30b7f2af" />


Add SonarQube Scanner installations 

Tools > SonarQube Scanner installations > Add SonarQube Scanner

```
Name - sonar-scanner
Install automatically - Install from Maven Central
Version - SonarQube Scanner 6.1.0.4477
```

<img width="839" alt="image" src="https://github.com/user-attachments/assets/420526c1-4d10-4e54-9f26-03b15bcdb1eb" />


Add Maven installation

Tools > Maven installation > Add Maven

```
Name - maven3
Install automatically - Install from Apache
Version = 3.9.8
```

<img width="818" alt="image" src="https://github.com/user-attachments/assets/fae60c40-0587-4425-b149-a2de207f67e3" />


Add Docker installation

Tools > Docker Installation > Add Docker

```
Name - docker
Install automatically - Download from docker.com
Docker version - latest
```

<img width="840" alt="image" src="https://github.com/user-attachments/assets/9cd820dc-b1c7-4456-8b10-1440898800ae" />


Apply & save

## To Create a Jenkins Pipeline to Checkout the Project

To create a new project named "techgame" with Pipeline.

General > Choose > Discard old builds

```
Strategy - Log Rotation
Max # of builds to keep - 2
Definition - Pipeline script
```

To create a credential in Jenkins console

Credentials > System > Global > Username and password

```
User name - kohlidevops
password - <git-token>
ID - git-cred
create
```

You can referr the Pipeline syntax to create script for example - git:Git

```
Sample Step - git:Git
Repository URL - https://github.com/kohlidevops/techgame.git
Branch - main
Credentials - <your cred>
Generate Pipeline script
```

Now your Checkout code could be 

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
    }
}
```

Apply and save - to run the build

The build has been succeeded.

You can check the source code in below path of Jenkins server

```
/var/lib/jenkins/workspace/techgame
```

<img width="907" alt="image" src="https://github.com/user-attachments/assets/11d37f2c-4064-4424-906d-7c56bdb462e6" />


## To Compile the Source code

For Maven Lifecycle you can refer - https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
    }
}
```

Run the build. Now the "target" directory should created in "/var/lib/jenkins/workspace/techgame" localtion once compiled.


## To run the Unit Test cases on Source code

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
    }
}
```

The build has been succeeded with Compile and run the unit test case.

## To Vulnerability on Source Code using Trivy

Refer - https://trivy.dev/v0.20.2/getting-started/installation/

SSH to Jenkins server and install Trivy 

```
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
trivy --version
```

After update the Trivy stage in Jenkins Pipeline

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Vulnerability Scan using Trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
    }
}
```

The build has been succeeded

<img width="906" alt="image" src="https://github.com/user-attachments/assets/d576a218-7f82-4868-8c75-ad0a12a17008" />

The trivy report is available in - /var/lib/jenkins/workspace/techgame


## To check Code Quality using SonarQube tool

- To generate the SonarQube token

Access the SonarQube Server > Login > Administration > Security > Token > Generate Tokens >

```
Name - sonar-token
Generate
//Keep this token safely
```

- To Configure the Sonar token in Jenkins credentials

Jenkins > Manage Jenkins > Credentials > System > Global > Add Credentials

```
Kind - Secret text 
Secret - ********* //Place that sonar-token value
ID - sonar-token
create
```

<img width="856" alt="image" src="https://github.com/user-attachments/assets/61348742-9a09-4814-84ec-a374d0e54ab3" />


- To add a SonarQube Server

Jenkins > Manage Jenkins > System > SonarQube installation

```
Name - sonar
Server URL - http://43.204.211.37:9000
Server authentication token - sonar-token //what you have created now
Apply & Save
```

After add the SonarQube analysis stage into Pipleine

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Vulnerability Scan using Trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=techgame -Dsonar.projectKey=techgame \
                -Dsonar.java.binaries=.'''
                }
            }
        }
    }
}
```

Apply & Save - To run the build. The build has been succeeded with Quality analysis on code

<img width="928" alt="image" src="https://github.com/user-attachments/assets/49b31cd3-1891-4108-8c03-d170ca600401" />


- To Create a Webhook in Sonar Server

Sonar server > Login > Administration > Configuration > Webhook > Create

```
Name - jenkins
URL - http://15.207.112.70:8080/sonarqube-webhook/
Create
```

- To add the Quality Gate Check

Once you added the Quality Gate check stage, then your code would be

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Vulnerability Scan using Trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=techgame -Dsonar.projectKey=techgame \
                -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('Wait For Quality Gate Status') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
    }
}
```

Apply & Save - To run the build.

<img width="851" alt="image" src="https://github.com/user-attachments/assets/4dfd2c5f-d5ff-4a20-98f6-baad649904f8" />


## To build the package using Maven tool

After adding the build package stage into the Jenkins pipeline

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Vulnerability Scan using Trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=techgame -Dsonar.projectKey=techgame \
                -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('Wait For Quality Gate Status') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build the Package') {
            steps {
                sh "mvn package"
            }
        }
    }
}
```

<img width="932" alt="image" src="https://github.com/user-attachments/assets/4c979aae-04af-41c8-9a66-ccaa80c70eda" />


After build the package stage, the jar file will be availablle in - /var/lib/jenkins/workspace/techgame/target/


## To Upload the Artifacts to the Nexus Repository

- To update the pom.xml in Source code

To change your Nexus repository URL in Distribution management of pom.xml

```
https://github.com/kohlidevops/techgame/blob/main/pom.xml
```

<img width="710" alt="image" src="https://github.com/user-attachments/assets/2fec5f74-aa85-4052-8703-9660db20e287" />


- To add a Maven Global Settings from the Managed files

Jenkins > Managed Jenkins > Managed files > Add a new config > Global Maven settings.xml > ID - global-settings > Next

```
ID - global-settings
Name - MyGlobalSettings
Comment - Global settings
Choose - Replace all
Content >

<server>
      <id>maven-releases</id>
      <username>admin</username>
      <password>admin@23</password>
    </server>
    <server>
      <id>maven-snapshots</id>
      <username>admin</username>
      <password>admin@23</password>
    </server>

submit
```

<img width="707" alt="image" src="https://github.com/user-attachments/assets/acc19707-43a0-452f-b308-e6a68ff5be14" />


- To add a Publish Artifact to the Nexus Repository Stage

Use Pipeline syntax to generate > Pipeline syntax

```
Sample step - withMaven: Provide Maven Environment
Maven - maven3
JDK - jdk17
Global Maven Settings Config - MyGlobalSettings
//These are the values what we have created before
Generate the pipeline script and apply this to the stage in Pipeline
```

Jenkins > Project > Configure > Pipeline

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Vulnerability Scan using Trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=techgame -Dsonar.projectKey=techgame \
                -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('Wait For Quality Gate Status') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build the Package') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish the Artifacts to the Nexus Repository') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                    }
                }
            }
        }
    }
```

After the build has been succeeded

<img width="946" alt="image" src="https://github.com/user-attachments/assets/9ccaff17-0c96-42b4-8662-1dc1bc466997" />

If you check with Nexus Repository > Maven Releases

<img width="731" alt="image" src="https://github.com/user-attachments/assets/2ecd9c30-cd38-4ab4-bcbb-d33cc865293c" />

Your package has been successfully released to Nexus repository


## To Build and Tag the Docker Image

- To Create a Dockerfile

To create a Dockerfile in Jenkins server with below location path

/var/lib/jenkins/workspace/techgame/

docker login -u latchudevops

sudo vi Dockerfile

```
FROM adoptopenjdk/openjdk11
EXPOSE 8080
ENV APP_HOME /usr/src/app
COPY target/*.jar $APP_HOME/app.jar
WORKDIR $APP_HOME
CMD ["java", "-jar", "app.jar"]
```

- To update the pom.xml to send snapshots to Nexus Snapshot

Open > pom.xml in Source code repo > edit > 0.04-SNAPSHOT

<img width="545" alt="image" src="https://github.com/user-attachments/assets/552035f9-ede6-4a81-9ee4-a49008cfed75" />


- To create a docker hub credentials in Jenkins credentials

Jenkins > Manage Jenkins > Credentials > System > Global > Add credential >

```
Kind - Username and Password
Username - latchudevops
Password - ******
ID - dock-cred
Create
```

- To generate the stage using Pipeline syntax

```
Sample step - withDockerRegistry: Sets up Docker registry endpoint
Registry credential - latchudevops //Select what you created now
Generate
```

After adding the stage

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Vulnerability Scan using Trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=techgame -Dsonar.projectKey=techgame \
                -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('Wait For Quality Gate Status') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build the Package') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish the Artifacts to the Nexus Repository') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                    }
                }
            }
        stage('To Build and Tag the Docker Image') {
            steps {
                scripts {
                    withDockerRegistry(credentialsId: 'dock-cred', toolName: 'docker') {
                        sh "docker build -t latchudevops/techgame:latest ."
                        
                    }
                }
            }
        }
        }
    }
```

Apply & save - To run the build - The build has been succeeded

<img width="917" alt="image" src="https://github.com/user-attachments/assets/8d46235a-dab9-4307-8acb-be1d8bc0cdc0" />

The snapshots are available in Nexus repository

<img width="707" alt="image" src="https://github.com/user-attachments/assets/dadc38c0-b717-4813-b76d-d6f446e9b142" />

The docker images are available in Jenkins server


## To Scan the Docker Image using Trivy tool

To add the docker scanning image stage into the Jenkins pipeline

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Vulnerability Scan using Trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=techgame -Dsonar.projectKey=techgame \
                -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('Wait For Quality Gate Status') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build the Package') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish the Artifacts to the Nexus Repository') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                    }
                }
            }
        stage('To Build and Tag the Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dock-cred', toolName: 'docker') {
                        sh "docker build -t latchudevops/techgame:latest ."
                        
                    }
                }
            }
        }
        stage('Docker Image Scanning using Trivy') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html latchudevops/techgame"
            }
        }
        }
    }
```

Apply and save - to run the build

<img width="953" alt="image" src="https://github.com/user-attachments/assets/4a7e0668-d2ea-4494-a6ab-28374abf5d26" />


If you check with Jenkins server for trivy report for image scanning

<img width="560" alt="image" src="https://github.com/user-attachments/assets/d92d1369-e989-4d7d-8787-16017784c364" />


## To Push the Docker Image to Docker Hub Registry

After adding the stage in pipeline

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Vulnerability Scan using Trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=techgame -Dsonar.projectKey=techgame \
                -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('Wait For Quality Gate Status') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build the Package') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish the Artifacts to the Nexus Repository') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                    }
                }
            }
        stage('To Build and Tag the Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dock-cred', toolName: 'docker') {
                        sh "docker build -t latchudevops/techgame:latest ."
                        
                    }
                }
            }
        }
        stage('Docker Image Scanning using Trivy') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html latchudevops/techgame"
            }
        }
        stage('To Push the Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dock-cred', toolName: 'docker') {
                        sh "docker push latchudevops/techgame:latest"
                        
                    }
                }
            }
        }
        }
    }
```

Apply & save - to run the build

<img width="899" alt="image" src="https://github.com/user-attachments/assets/7140d311-ab65-4234-af43-4ad7e960b91e" />

Once pushed the image, you can check in Docker Hub registry

<img width="907" alt="image" src="https://github.com/user-attachments/assets/11b71e83-c7d6-4c70-a239-ef52bc5c00c5" />


## To deploy the application to a Kubernetes Cluster Environment


- To Create a namespace

SSH to master node and create a new namespace named webapps

```
kubectl get ns
kubectl create ns webapps
kubectl get ns
```

- To Create a ServiceAccount

```
mkdir deploy
cd deploy
nano webapps-serviceaccount.yaml

https://github.com/kohlidevops/DevOpsProjects/blob/main/webapps-serviceaccount.yaml

kubectl get serviceaccount
kubectl apply -f webapps-serviceaccount.yaml
kubectl get serviceaccount -n webapps
```

<img width="674" alt="image" src="https://github.com/user-attachments/assets/da23eba9-03d4-43c1-9a45-58cc289d5ea3" />


- To Create a Role

To create Role and this role should be created within webapps namespace

```
cd deploy
nano webapps-role.yaml

https://github.com/kohlidevops/DevOpsProjects/blob/main/webapps-role.yaml

kubectl apply -f webapps-role.yaml
kubectl get role -n webapps
```

<img width="602" alt="image" src="https://github.com/user-attachments/assets/8decc00b-9a5f-4eef-a1a8-50c24395892a" />


- To Create a RoleBinding to apply a Role

```
cd deploy
nano webapps-rolebinding.yaml

https://github.com/kohlidevops/DevOpsProjects/blob/main/webapps-rolebinding.yaml

kubectl apply webapps-rolebinding.yaml
kubectl get RoleBinding -n webapps
```

<img width="656" alt="image" src="https://github.com/user-attachments/assets/609ca807-cd8b-4f56-a73c-57ecdbc0fa02" />

- To Create a Secret to access the cluster from jenkins

```
cd deploy
nano webapps-secret.yaml

https://github.com/kohlidevops/DevOpsProjects/blob/main/webapps-secret.yaml

kubectl apply -f webapps-secret.yaml
kubectl get secrets -n webapps
```

- To get or describe the secret

To get the secret and assign this secret to Jenkins server to grant access the Kubernetes Cluster

```
kubectl get secrets -n webapps
kubectl describe secret mysecretname -n webapps
```

<img width="943" alt="image" src="https://github.com/user-attachments/assets/1dc9c0a5-a6ee-4425-aa48-44d336ac3cf4" />


- To create a credentials for kubernetes token

Jenkins > Manage Jenkins > Credentials > System > Global > Add new

```
Kind - Secret text
Secret - //k8s-token
ID - secret
Create
```

<img width="913" alt="image" src="https://github.com/user-attachments/assets/2fb53b6b-7f9c-4293-af0a-55881aae3e14" />


- To get the kuberenets service endpoint

SSH to master node

```
cat ~/.kube/config
//You can collect from here
```

<img width="616" alt="image" src="https://github.com/user-attachments/assets/6cb6e8e8-81df-4a31-8f38-bad348b0a3e0" />


- To add a Kubernetes Deployment stage in Jenkins

Refer Pipeline syntax to complete this stage

```
Sample step - withKubeConfig: Configure Kubernetes CLI (kubectl)
Credentials - k8s-cred
Kubernetes server endpoint - https://172.31.10.77:6443
Cluster name - Kubernets
Namespace - webapps
```

Generate Pipeline script

```
withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.10.77:6443') {
    // some block
}
```

- To create a deployment and service manifiest in Source code repo

```
https://github.com/kohlidevops/DevOpsProjects/blob/main/webapps-deployment-service.yaml
```

After you adding the deployment stage

```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout the Project from the GitHub to Jenkins Server') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/kohlidevops/techgame.git'
            }
        }
        stage('Compile the Source Code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test cases on Source Code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Vulnerability Scan using Trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=techgame -Dsonar.projectKey=techgame \
                -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('Wait For Quality Gate Status') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build the Package') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish the Artifacts to the Nexus Repository') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                    }
                }
            }
        stage('To Build and Tag the Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dock-cred', toolName: 'docker') {
                        sh "docker build -t latchudevops/techgame:latest ."
                        
                    }
                }
            }
        }
        stage('Docker Image Scanning using Trivy') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html latchudevops/techgame"
            }
        }
        stage('To Push the Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dock-cred', toolName: 'docker') {
                        sh "docker push latchudevops/techgame:latest"
                        
                    }
                }
            }
        }
        stage('Deploy the Docker Image to the Kubernetes Cluster') {
            steps {
            withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.10.77:6443') {
                sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }
        stage('To Verify the Deployment and Service') {
            steps {
            withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.10.77:6443') {
                sh "kubectl get pods -o wide -n webapps"
                sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
```

- To Install kubectl, kubelet and kubeadm in Jenkins Server

SSH to Jenkins and execute below script

```
nano k8s-install.sh

https://github.com/kohlidevops/DevOpsProjects/blob/main/k8s-install.sh

sudo chmod +x k8s-install.sh
sudo sh k8s-install.sh
```


- To build the Jenkins

To start the build and check the console output and status of the pods in K8 master


<img width="953" alt="image" src="https://github.com/user-attachments/assets/4c88e2a9-6e92-49a0-8b6e-8b3402c87786" />


<img width="895" alt="image" src="https://github.com/user-attachments/assets/1d1cf6e7-e4ad-47b3-ac9c-cca4091b17ca" />


If you are facing any issues with "Failed to setup network" - It can be weave network plugins issues. You can check the pods

```
kubectl get pods -n kube-system -o wide | grep weave
kubectl rollout restart ds weave-net -n kube-system
kubectl get pods -n kube-system -o wide | grep weave
```

Now you can access the application using Worker node IP with Port numer


<img width="933" alt="image" src="https://github.com/user-attachments/assets/301d424c-59af-4f34-a56f-c3f88e492c33" />


## To Monitoring with Prometheus and Grafana

- To download and install Prometheus

SSH to Jenkins server

Refer - https://prometheus.io/download/

```
mkdir monitoring
cd monitoring
wget https://github.com/prometheus/prometheus/releases/download/v2.53.3/prometheus-2.53.3.linux-amd64.tar.gz
tar -xvzf prometheus-2.53.3.linux-amd64.tar.gz
cd prometheus-2.53.3.linux-amd64/
./prometheus &
```

You can access the Prometheus - IP:9090

<img width="863" alt="image" src="https://github.com/user-attachments/assets/3a8a6576-28b9-41bb-b380-4f389349e7aa" />

- To download and install Grafana

Refer - https://grafana.com/grafana/download/11.1.3

SSH to Jenkins server

```
cd monitoring
sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_11.1.3_amd64.deb
sudo dpkg -i grafana-enterprise_11.1.3_amd64.deb
### NOT starting on installation, please execute the following statements to configure grafana to start automatically using systemd
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable grafana-server
### You can start grafana-server by executing
sudo /bin/systemctl start grafana-server
```

You can access the Grafana - IP:3000

<img width="746" alt="image" src="https://github.com/user-attachments/assets/5af313fa-dcc1-42c7-943d-40c82c3b38bf" />

Default username and password - admin and to change the password


- To download and install Blackbox Exporter

Refer - https://prometheus.io/download/#blackbox_exporter

SSH to Jenkins

```
cd monitoring
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar -xvzf blackbox_exporter-0.25.0.linux-amd64.tar.gz 
cd blackbox_exporter-0.25.0.linux-amd64
./blackbox_exporter &
```

You can access the Blackbox Exporter - IP:9115

<img width="625" alt="image" src="https://github.com/user-attachments/assets/850cb95d-0408-49a6-a8e6-d7b3cf70ec04" />


- To edit the Prometheus yaml and add the neccessary entry on it


Refer - https://github.com/prometheus/blackbox_exporter

Copy/Paste the below content from github to jenkins server - Prometheus yaml (end of file)

```
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://prometheus.io   # Target to probe with https.
        - http://example.com:8080 # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.
```

Do the neccessary changes and restart the Prometheus


<img width="694" alt="image" src="https://github.com/user-attachments/assets/fb825492-37d1-459f-b917-9ddba4ab5e41" />


- To restart Prometheus

```
pgrep prometheus
kill 10659
./prometheus &
```

- To access the Prometheus dashboard

Prometheus > Status > Targets

<img width="914" alt="image" src="https://github.com/user-attachments/assets/cf27b152-cb46-4c95-94bc-d99da403900a" />

If you check with Blackbox exporter, then you can see the various probes

<img width="518" alt="image" src="https://github.com/user-attachments/assets/55ff690a-72cb-4c6e-9d0b-87a2791e7ba3" />


- To visulaize the application data using Grafana

To login the Grafana Dashboard and choose the connections from Left side panel

Connections > Data sources > Add Data source > Prometheus

```
Name - Prometheus
Prometheus Server URL - http://15.207.100.247:9090
Save and Test
```

Then Import the Dashboard

![image](https://github.com/user-attachments/assets/6704c4bb-9c1d-490b-ac51-80c3c8d8ec15)

Now have to give grafana dashboard ID

you can find the ID - https://grafana.com/grafana/dashboards/7587-prometheus-blackbox-exporter/

Copy ID to clipboard and Load the Dashboard > signcl-prometheus > choose > prometheus > Import

After Import, you can see the visualization in Grafana

<img width="934" alt="image" src="https://github.com/user-attachments/assets/7e9e14fc-a8d1-4215-854e-ff48be1fd993" />


- To monitor the Server Metrics using Prometheus with Node Exporter

Download and install the Node Exporter in Jenkins

Install Prometheus plugins in Jenkins Dashboard

Jenkins > Manage Jenkins > Plugins > Available > Prometheus Metrics > Install


SSH to Jenkins Server

```
cd monitoring
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar -xvzf node_exporter-1.8.2.linux-amd64.tar.gz
cd node_exporter-1.8.2.linux-amd64/
./node_exporter &
```

To edit the Prometheus yaml file to add the server metrics

```
cd prometheus-2.53.3.linux-amd64
sudo nano prometheus.yml

//server metrics content

 - job_name: 'node_exporter'
   static_configs:
     - targets: ['15.207.100.247:9100']   //node exporter which is in jenkins server
 - job_name: 'jenkins'
   metrics_path: '/prometheus'
   static_configs:
     - targets: ['15.207.100.247:8080']   //jenkins server
```

To restart the Prometheus

```
pgrep prometheus
kill 11959
./prometheus &
```

My Prometheus yaml looks like

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['15.207.100.247:9100']
  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['15.207.100.247:8080'] 
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://prometheus.io   # Target to probe with https.
        - http://13.235.49.255:32718 # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 15.207.100.247:9115  # The blackbox exporter's real hostname:port.
```

After restarting prometheus, I can able to see the metrics in Prometheus dashboard

<img width="921" alt="image" src="https://github.com/user-attachments/assets/9f287aad-4820-40d4-95a4-0a762f20ea20" />


To visualize the metrics using Grafana1860

First collect the ID from the node exporter with grafana - https://grafana.com/grafana/dashboards/1860-node-exporter-full/

Grafana Dashboard > New Dashboard > Import Dashboard > ID > Load > Choose > Promethues > Import

Now you can able to see the Jenkins Server metrics as Grafana visualization

<img width="932" alt="image" src="https://github.com/user-attachments/assets/2cf10b2e-0b83-4cb9-af45-217a23a9623d" />

You can see the two dashboard from here - one is NodeExporter and another one is BlackboxExporter

<img width="821" alt="image" src="https://github.com/user-attachments/assets/867cd97f-f621-475c-af02-290aab6d8df3" />


