# To Setup DevOps Project Using Terraform, Ansible, Jenkins, Maven, SonarQube, Docker, Helm Charts, Kubernetes, Promotheus and Grafana

## Pre-requisites

1. AWS CloudShell

2. Install and configure AWS CLI on Cloudshell

Refer - https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

3. Install Terraform on Cloudshell

Refer - https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli

```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform
terraform -help
```

4. Github repository

source code repo - https://github.com/kohlidevops/tweet-trend-new

lab code repo - https://github.com/kohlidevops/devops-workshop

## Terraform to provisioning resources

**1. To create a simple Ec2 instance using terraform**

https://github.com/kohlidevops/devops-workshop/blob/main/terraform_code/V1-EC2.tf

To change the values as per your environment and run the below command

```
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy  #once your checked with your aws console, then you can destroy it
```

<img width="793" alt="image" src="https://github.com/user-attachments/assets/da54de5a-82ec-4ad9-bff5-4a99af570b03" />

**2. To create a Ec2 instance along with Security group**

https://github.com/kohlidevops/devops-workshop/blob/main/terraform_code/V2-EC2.tf

```
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy  #once your checked with your aws console, then you can destroy it
```

**3. To create a VPC and Ec2 instance along with Security group**

https://github.com/kohlidevops/devops-workshop/blob/main/terraform_code/V3-EC2-With_VPC.tf

```
terraform init
terraform validate
terraform plan
terraform apply
```

Dont destroy this template

**4. To create a 3 more Ec2 instances using for each loop**

https://github.com/kohlidevops/devops-workshop/blob/main/terraform_code/V4-EC2-With_VPC_for_each.tf

To change the AMI as ubuntu-22

```
terraform init
terraform validate
terraform plan
terraform apply
```

Now exisiting instance will be deleted then create a new 3 Ec2 instances

<img width="797" alt="image" src="https://github.com/user-attachments/assets/6bdd8b3e-631e-4645-95cd-0a9d4f212352" />

## To Setup Ansible and install jenkins and maven

**1. Install Ansible on ubuntu machine**

Refer - https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu

```
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
ansible --version
```

**2. To setup connectivity between ansible and other two machines**

For this, we need to upload the private key pair of jenkins and slave machines to ansible machine. Then create ansible host file

```
cd /opt/
//keypair should be available here
//create a ansible hosts file
nano hosts

#################################################
[jenkins-master]
10.1.1.211

[jenkins-master:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/opt/devopstep1.pem

[jenkins-slave]
10.1.1.182

[jenkins-slave:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/opt/devopstep1.pem
#################################################

```

_To ping the jenkins master from ansible_

```
ansible all -i hosts -m ping
```

<img width="947" alt="image" src="https://github.com/user-attachments/assets/33b213bf-d07b-4785-a677-75a9e73ba5a5" />


**3. To install jenkins on jenkins-master machine using ansible playbook**

Refer - https://pkg.origin.jenkins.io/debian-stable/

https://github.com/kohlidevops/devops-workshop/blob/main/Ansible/jenkins-master-setup.yaml

_To run the jenkins-master-setup file_

```
ansible-playbook -i /opt/hosts jenkins-master-setup.yaml --check
ansible-playbook -i /opt/hosts jenkins-master-setup.yaml
```

<img width="943" alt="image" src="https://github.com/user-attachments/assets/92613cd6-36c9-416b-837c-98623b4307f5" />


_To access the jenkins portal_

<img width="796" alt="image" src="https://github.com/user-attachments/assets/5de5dca4-3478-42fa-b391-b9b5b2b49df1" />


_Install the suggested plugins and create a first admin user_

<img width="830" alt="image" src="https://github.com/user-attachments/assets/64c33f92-5e83-40bc-abaf-f375f612c8df" />

<img width="773" alt="image" src="https://github.com/user-attachments/assets/5dd647e1-7daa-45a1-913e-d3c33d7e78b0" />

let's save and finish to continue the jenkins

**4. To install a Maven on jenkins-slave machine using ansible playbook**

Refer - https://dlcdn.apache.org/maven/maven-3/

https://github.com/kohlidevops/devops-workshop/blob/main/Ansible/v1-jenkins-slave-setup.yaml

_To run the v1-jenkins-slave-setup.yaml_

```
ansible-playbook -i /opt/hosts v1-jenkins-slave-setup.yaml --check
ansible-playbook -i /opt/hosts v1-jenkins-slave-setup.yaml
```

<img width="934" alt="image" src="https://github.com/user-attachments/assets/e079e066-ded5-45d8-a1d2-668800e512d3" />

_SSH to jenkins-slave machine_

```
sudo -i
cd /opt/apache-maven-3.9.4/bin
./mvn --version
```

<img width="656" alt="image" src="https://github.com/user-attachments/assets/5d2c94bd-fd16-47ce-a932-0e97b2dc275f" />

## Jenkins Configuration

**1. To add a Maven server credential to the jenkins portal**

Login to Jenkins portal > Dashboard > Manage Jenkins > Credentials > System > Global credentials (unrestricted) > New credentials > SSH username and private key > create

```
ID - maven-server-cred
Username - ubuntu
Private key - Enter directly
```

**2. To add a Maven server as a Jenkins Slave**

Dashboard > Manage Jenkins > Nodes > New Node > create

```
Node name - maven-slave
Type - Permanent Agent
Number of executors: 3
Remote root directory: /home/ubuntu/jenkins
Labels: maven
Usage: Use this node as much as possible
Launch method: Launch agents via SSH
Host: <Private_IP_of_Slave>
Credentials: <Jenkins_Slave_Credentials>
Host Key Verification Strategy: Non verifying Verification Strategy
Availability: Keep this agent online as much as possible
```

If you check the logs of slave machine

<img width="844" alt="image" src="https://github.com/user-attachments/assets/d5fe6f9f-e795-4df5-a993-414df2d194ae" />

<img width="934" alt="image" src="https://github.com/user-attachments/assets/8692740b-5f7e-4e50-a856-b541c3ca453b" />

If you check with jenkins-slave machine through SSH

<img width="461" alt="image" src="https://github.com/user-attachments/assets/ee0dcc12-56ea-4ab8-87cd-5624a771d105" />


**3. To test a simple project with jenkins-slave**

Create a test-job with Freestyle project

Configure > Choose > Restrict where this project can be run > 

```
Label Expression - maven
Build Steps - Execute Shell - echo "Hello, I'm a Slave machine" >> /home/ubuntu/maven.txt
```

Apply & save - Run the build.

<img width="897" alt="image" src="https://github.com/user-attachments/assets/80901cf3-24c8-452e-8086-02a2b58ec096" />

If SSH to maven machine

<img width="493" alt="image" src="https://github.com/user-attachments/assets/f4a4f4bc-1a4c-4745-b9cd-f733160e1df2" />



**4. To test a sample Hello world project with Pipeline in jenkins**

Create a ttrend-job with Pipeline

Pipeline > pipeline script > Choose Hello World and do some changes like below

```
pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

Apply & Save then build the job.

<img width="902" alt="image" src="https://github.com/user-attachments/assets/9f3b0e86-4509-4531-ac38-fa9f25cc4be7" />


**5. To add the Checkout stage in to the ttrend-job pipeline**

```
pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    stages {
        stage('Clone-code') {
            steps {
                git branch: 'main', url: 'https://github.com/kohlidevops/tweet-trend-new.git'
            }
        }
    }
}
```

Apply & save - Then start the build

<img width="773" alt="image" src="https://github.com/user-attachments/assets/a55435e1-62b7-4c6b-a63b-2075949a53e2" />


<img width="658" alt="image" src="https://github.com/user-attachments/assets/fe30b8b0-df3e-4dce-9313-ad7fbda45219" />

**6. To test a Hello World project with Jenkinsfile in Jenkins**

Create a Jenkinsfile in your source code repository add the below code then to start the build

<img width="900" alt="image" src="https://github.com/user-attachments/assets/7d1544f3-2c8b-4892-923f-6bce1f12e3e7" />


```
pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    stages {
        stage('Clone-code') {
            steps {
                git branch: 'main', url: 'https://github.com/kohlidevops/tweet-trend-new.git'
            }
        }
    }
}
```

Go to jenkins portal > ttrend-job > Configuration > Pipeline

```
Definition - Pipeline script from SCM
SCM - Git
Repository URL - https://github.com/kohlidevops/tweet-trend-new.git
Branch Specifier - */main
Script Path - Jenkinsfile
```

Apply & Save - Then build the job


<img width="778" alt="image" src="https://github.com/user-attachments/assets/0430b6e7-1479-42b1-b51b-bcdbc3286438" />


**7. To add a build stage into Jenkinsfile**

To open the Jenkinsfile and update the below code

```
pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
}
    stages {
        stage('build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
```

I remove the clone-code stage as it doesn't require, because Jenkins will clone when it is going to read the Jenkinsfile.

Now trigger the build and check.












