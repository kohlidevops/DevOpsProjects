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

Now trigger the build and check. // Sometimes build getting failed due to the jdk-17 version. Try to downgrade to jdk-11 in maven server using below commands.

```
sudo apt install openjdk-11-jdk
sudo update-alternatives --config java
```

<img width="801" alt="image" src="https://github.com/user-attachments/assets/dbf782e7-a91f-4329-8fe9-db4febe530e2" />

**8. To add a GitHub credentials to Jenkins**

GitHub source code repo > Profile > Settings > Developer Settings > Personal Access Tokens > Tokens (Classic) > New Personal Access Token > Generate Token

<img width="752" alt="image" src="https://github.com/user-attachments/assets/60943a5d-b68d-4088-9510-df3f7e82f696" />

Navigate to Jenkins console > Manage Jenkins > Credentials > System > Global > Add Credentials > User name with password

```
Kind - Username and password
Username - kohlidevops
Password - Jenkins-Token
ID - Github_Cred
Save
```

<img width="851" alt="image" src="https://github.com/user-attachments/assets/087cb61f-36e1-4405-8d58-b260eb164612" />

_To update the Github creds to the ttrend-job project_

ttrend-project > configure > update in SCM

<img width="646" alt="image" src="https://github.com/user-attachments/assets/f32c70f6-f0b8-4d9b-afd2-aa3e5e35915e" />

Apply & Save - Build the project

<img width="755" alt="image" src="https://github.com/user-attachments/assets/92017ff4-2139-480d-bbb0-8b4509b8022a" />

**9. To Setup a Multi-branch Pipeline**

Jenkins console> New Item > ttrend-multibranch > Multibranch Pipeline > create

```
Branch Sources > Git
Project Repository > https://github.com/kohlidevops/tweet-trend-new.git
Credentials > Choose your creds
Build Configuration > Mode > by Jenkinsfile
Script Path > Jenkinsfile
Apply & Save
```

Once you saved, then it will automatically discover the "main" branch and start the build. If you create a new branch from main branch in Github, then it will start the build once you choose "Scan Multibranch Pipeline Now" in Jenkins console.

<img width="907" alt="image" src="https://github.com/user-attachments/assets/7db19f60-2ed0-42ca-a64d-825bcdca2feb" />

Now create a new branch in Github

<img width="845" alt="image" src="https://github.com/user-attachments/assets/9980a546-e27a-4b8e-9351-dc53d9bae81f" />

If you "Scan Multibranch Pipeline Now", then you can find the dev branch in Jenkins and as well its getting initiated the build.

<img width="910" alt="image" src="https://github.com/user-attachments/assets/3b935eee-73b6-4998-923a-afea7c7c82f9" />

It means, when the branch has Jenkinsfile it will auto-discover in the Jenkins console, as well initiate the build. If there is no Jenkinsfile in any branch it will not discover by Jenkins console.

<img width="918" alt="image" src="https://github.com/user-attachments/assets/d0d047f4-a3ee-4850-b522-f0651de1de83" />


**10. To Setup Webhook in Github**

Why Webhook? - When any new changes occur in the Source code repository, then this webhook will help jenkins to trigger the build automatically.

_a. Install "multibranch scan webhook trigger" plugin_

From dashboard > manage jenkins > manage plugins > Available Plugins > Search for "Multibranch Scan webhook Trigger" plugin > install it

_b. Go to multibranch pipeline job _

job > configure > Scan Multibranch Pipeline Triggers > Scan by webhook > Trigger token: <latchu-scan>

<img width="880" alt="image" src="https://github.com/user-attachments/assets/1a2e164f-cca1-4afd-a126-3c2cc8febb12" />

apply > save

_c. Add webhook to GitHub repository _

Github repo (your source code repo) > settings > webhooks > Add webhook

```
Payload URl: http://43.204.22.22:8080/multibranch-webhook-trigger/invoke?token=latchu-scan
Content type: application/json
Which event would you like to trigger this webhook: just the push event
```

<img width="822" alt="image" src="https://github.com/user-attachments/assets/db2815ce-0917-4652-98f0-a531a4c83176" />

Do some changes in both "main" and "dev" branch - Then it should auto trigger the build through webhooks

<img width="832" alt="image" src="https://github.com/user-attachments/assets/6a01bc72-a318-4628-b275-3085cbb263d7" />


## SonarQube Integration with Jenkins

**1. To Setup SonarQube account and add Sonar credentials to the Jenkins**

_a. Create Sonar cloud account_

https://sonarcloud.io > Login with Github > Authorize the account

<img width="937" alt="image" src="https://github.com/user-attachments/assets/f429efc5-ce09-4301-a832-5d1653943961" />

_b. Generate an Authentication token on SonarQube_     

Account > my account > Security > Generate Tokens

<img width="653" alt="image" src="https://github.com/user-attachments/assets/f87342d4-b711-4291-a3cc-08875c38a8cb" />

_c. On Jenkins create credentials _   

Manage Jenkins > manage credentials > system > Global credentials > add credentials

```
Credentials type: Secret text
ID: sonarqube-key
save
```

_d. Install SonarQube plugin_

Manage Jenkins > Available plugins > Search for > sonarqube scanner > install it

_e. Configure sonarqube server_

Manage Jenkins > Configure System > sonarqube server  > Sonarqube installation > Add Sonarqube server    

```
Name: sonar-server    
Server URL: https://sonarcloud.io
Server authentication token: sonar-cred
```

_f. Configure sonarqube scanner_

Manage Jenkins > Global Tool configuration > SonarQube Scanner installations > Add sonarqube scanner

```
Sonarqube scanner: sonar-scanner
Version - SonarQube Scanner 4.8.0.2856
Apply & Save
```

<img width="832" alt="image" src="https://github.com/user-attachments/assets/6aabd7ca-6e7d-4077-8aad-128ff775620f" />

**2. To create a new sonar properties**

Go to https://sonarcloud.io/

My account > Organizations > Create > You can create one manually 

<img width="952" alt="image" src="https://github.com/user-attachments/assets/e8a0914f-746a-4aa8-b7c1-b1573f0dcf1c" />

Create an Organization

```
Name - latchu
key - latchu
Choose a plan - Free
Create organization
```

<img width="802" alt="image" src="https://github.com/user-attachments/assets/6ce88b7c-19e4-4aad-9d9c-71e6df08532b" />

To analyze a new project

```
Organization - latchu
Display Name - ttrend
Project key - latchu_ttrend
Project visibility - Public
Number of days - 30
Create Project
```

_To create a file named "sonar-project.properties" in your source code repository with below contents_

```
sonar.verbose=true
sonar.organization=latchu
sonar.projectKey=latchu_ttrend
sonar.projectName=ttrend
sonar.language=java
sonar.sourceEncoding=UTF-8
sonar.sources=.
sonar.java.binaries=target/classes
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

<img width="810" alt="image" src="https://github.com/user-attachments/assets/70dacd84-f82d-4c65-92cd-4b00c926ef31" />

To refer below link to get a sonar stages for jenkins file

https://docs.sonarsource.com/sonarqube-server/10.1/analyzing-source-code/scanners/jenkins-extension-sonarqube/

```
stage('SonarQube analysis') {
environment {
	scannerHome = tool 'sonar-scanner'
}
steps {
	withSonarQubeEnv('sonarqube-server') {  
      sh "${scannerHome}/bin/sonar-scanner"
    }
}
```

Your Jenkinsfile should like below once you updated the sonarqube stage

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
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'sonar-scanner'
                }
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
}
```


