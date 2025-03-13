# To Promote Staging to Production ECS using (Continuous Integration and Continuous Delivery) Jenkins, Nexus, SonarQube Analysis, QualityGate Status Check, Docker, Amazon Elastic Container Registry (ECR), Amazon Elastic Container Service (ECS) and Slack Channel (Notification System)


![image](https://github.com/user-attachments/assets/3d5dad9a-9284-418d-a2b9-05148e413bef)



## To Achieve the CICD Pipeline


**1. Maven Build -> Compiles the project and packages the code into .war or .jar**

**2. Maven Test -> Executes automated test cases to check if the code is functioning correctly**

**3. Maven CheckStyle -> Checks the code for style or formatting issues based on predefined rules**

**4. SonarQube Analysis -> Analyzes the code for bugs, vulnerabilities, and code smells (static code analysis) using SonarQube**

**5. Quality Gate -> A decision point based on the results of SonarQube analysis. It ensures that the code meets specific quality standards before proceeding**

**6. Nexus Repository -> After successful testing, the .war file is uploaded to the Nexus repository**

**7. Build App Docker Image -> The application is packaged into a Docker image by reading the Docker file**

**8. Upload App Image to ECR ->The Docker image is pushed to a ECR registry**

**9. Staging ECS -> To deploy the Docker image into Staging ECS Cluster**

**10. Production ECS -> To deploy the Docker image into Production ECS Cluster once staging branch is merged to production branch in GitHub repository**

**11. Slack Channel -> To send the Status of CICD Pipeline**


## Jenkins, Nexus and SonarQube Setup

Launch Ubuntu22 EC2 instance with t3.medium and execute the below script to install jenkins

```
https://github.com/kohlidevops/DevOpsProjects/blob/main/Jenkins-Setup.sh
```

Launch CentOS-9 EC2 instance with t3.medium and execute below script to install Nexus

```
https://github.com/kohlidevops/DevOpsProjects/blob/main/Nexus-Setup.sh
```

Launch Ubuntu22 EC2 instance with t3.medium and execute the below script to install SonarQube

```
https://github.com/kohlidevops/DevOpsProjects/blob/main/SonarQube-Setup.sh
```

## Configure Jenkins

SSH to instance

```
sudo -i
systemctl status jenkins.service
```

- To access the Jenkins dashboard with port-8080 and install the suggested plugins


![image](https://github.com/user-attachments/assets/2bb1fba9-a945-4ff1-af27-6805ec1d6700)


To install a below plugins

```
Maven Integration
Github Integration
Nexus Artifact Uploader
Sonarqube Scanner
Slack Notification
Build Timestamp
```

- To access the Nexus dashboard with port-8081 and configure it


![image](https://github.com/user-attachments/assets/63cb6c26-83e1-4f6b-aaeb-b9226b704c63)


To create a repositories with maven2(hosted) and name it as devops-release, version policy - Releases and create it

To create a repositories with maven2(proxy) and name it as devops-central, Remote storage - (https://repo1.maven.org/maven2/) and create it

To create a repositories with maven2(hosted) and name it as devops-snapshots, Version policy - Snapshots and create it

To create a repositories with maven2(group) and name it as devops-group, add the member to the group like below and create it

![image](https://github.com/user-attachments/assets/7ac327cf-a593-48a7-a39f-da9a48a334a5)


- To access the Sonarqube dashboard with port-80

Default username is admin and password is admin


![image](https://github.com/user-attachments/assets/d636b9a1-e883-4570-900a-fc880f35c223)


## Git setup in Local

Local machine > Gitbash

```
git config --global user.email "YOUR Email Address"
git config --global user.name "YOUR Username"
cd
cd ~/.ssh
ls
ssh-keygen  //Enter your GitHub account name
ls
cat PublicKey.pub //To store this in Github > Settings > SSH and GPG keys > Add SSH key > Authentication key > Place the public key > save
# Create ssh config file for GitHub account
vim ~/.ssh/config
Host github.com-kohlidevops
  User git
  IdentityFile ~/.ssh/kohlidevops
  HostName github.com

# Clone source code
cd
mkdir PathToCloneRepo
cd PathToCloneRepo
git clone git@github.com-kohlidevops:kohlidevops/vprofile-project.git
git checkout ci-jenkins

#Do some change and commit to check
vim README.md
git add .
git commit -m "first commit"
git push origin ci-jenkins
```

## To configure tools & credentials

Jenkins > Manage Jenkins > Tools > Add JDK

```
Name - OracleJDK11
JAVA_HOME - /usr/lib/jvm/java-1.17.0-openjdk-amd64

#Add one more JDK
#You can install in Jenkins server - apt-get install openjdk-21-jdk -y
#ls -lh /usr/lib/jvm/

Name - JDK21
JAVA_HOME - java-1.21.0-openjdk-amd64
```

Jenkins > Manage Jenkins > Tools > Add Maven

```
Name - MAVEN3.9
Version - 3.9.9
Apply & Save
```

## To add the Nexus credentials in Jenkins

Jenkins > Manage Jenkins > Credentials > System > Global > New > Username and Password

```
Username - admin
Password - ******
ID - nexuslogin
Save
```

![image](https://github.com/user-attachments/assets/c1c5aa8b-4ecb-4791-bef3-0917134e9c2e)


## To add the git credentail in Jenkins

Jenkins > Manage Jenkins > Credentials > System > Global > New > SSH Username with Private key

```
ID - gitlogin
Username - git
Private key - //Copy paste your git private key which is created using ssh-keygen
Then create
```

## To Create a new project in Jenkins

Jenkins > Add new item > Pipeline

```
Definition - Pipeline script from SCAM
SCM - Git
Repositories - git@github.com:kohlidevops/vprofile-project.git
Credentials - Add your git
Branch Specifier  - ci-jenkins
Script Path - Jenkinsfile
Apply & save
```

SSH to Jenkins server to hostkey verification

```
sudo -i
su - jenkins
ssh -T git@github.com
cat .ssh/known_hosts
```

To add the Jenkinsfile in Github repo with ci-jenkins branch

```
pipeline {
	agent any
	tools {
		jdk "OracleJDK11"
                maven "MAVEN3.9"
		}
	environment {
		SNAP_REPO = 'devops-snapshots'
                NEXUS_USER = 'admin'
                NEXUS_PASS = 'admin'
                RELEASE_REPO = 'devops-release'
                CENTRAL_REPO = 'devops-central'
                NEXUSIP = '172.31.1.225'
                NEXUSPORT = '8081'
                NEXUS_GRP_REPO = 'devops-group'
                NEXUS_LOGIN = 'nexuslogin'
		}
	stages{
		stage('BUILD'){
			steps {
				sh 'mvn -s settings.xml -DskipTests install'
				}
		}
	}
}
```

## To add the Jenkins Webhook, Archive, Test and Checkstyle analysis

Github > Your Project > Settings > Webhook > Add Webhook

```
Payload URL - http://13.201.75.108:8080/github-webhook/     //Jenkins IP
Content type - application/json
Event - Just the push event
```

Jenkins > Project > configure > Triggers > GitHub hook trigger for GITScm polling > Apply & Save


Github > Jenkinsfile

```
pipeline {
	agent any
	tools {
		jdk "OracleJDK8"
                maven "MAVEN3.9"
		}
	environment {
		SNAP_REPO = 'devops-snapshots'
                NEXUS_USER = 'admin'
                NEXUS_PASS = 'admin'
                RELEASE_REPO = 'devops-release'
                CENTRAL_REPO = 'devops-central'
                NEXUSIP = '172.31.1.225'
                NEXUSPORT = '8081'
                NEXUS_GRP_REPO = 'devops-group'
                NEXUS_LOGIN = 'nexuslogin'
		}
	stages{
		stage('BUILD'){
			steps {
				sh 'mvn -s settings.xml -DskipTests install'
				}
			post {
				success {
					echo "Archiving"
					archiveArtifacts artifacts: '**/*.war'
						}
					}
				}
		stage('Test') {
			steps {
				sh 'mvn -s settings.xml test'
			}
		}
		stage('Checkstyle Analysis') {
			steps {
				sh 'mvn -s settings.xml checkstyle:checkstyle'
			}
		}
	}
}
```

Once you commited, the build has been triggered automatically.


## To Code Analysis with Sonarqube

To generate the token in Sonarqube server

Sonarqube dashboard > Login > Administration > Security > Users > Adminstrator > Generate token > name it as jenkins > generate

To add the sonarwube credential in Jenkins 

Jenkins dashboard > Manage Jenkins > Credentials > System > Global > Add new

```
Kind - Secret text
Secret - <your-token>
ID - sonartoken
create
```

To add the sonar server in Jenkins tools

Jenkins > System > SonarQube installations > Add Sonarqube 

```
Name - sonarserver
Server URL - http://172.31.6.160
Server authentication token - sonartoken
apply & save
```

To add the sonar scanner in tools

Jenkins > Tools > SonarQube Scanner installations > Add SonaQube Scanner

```
Name - sonarscanner
Choose - Install Automatically
Choose - Install from Maven central
Version - SonarQube Scanner 4.7.0.2747
Apply & save
````

Github > Jenkinsfile

```
pipeline {
	agent any
	tools {
		jdk "OracleJDK8"
                maven "MAVEN3.9"
		}
	environment {
		SNAP_REPO = 'devops-snapshots'
                NEXUS_USER = 'admin'
                NEXUS_PASS = 'admin@23'
                RELEASE_REPO = 'devops-release'
                CENTRAL_REPO = 'devops-central'
                NEXUSIP = '172.31.9.243'
                NEXUSPORT = '8081'
                NEXUS_GRP_REPO = 'devops-group'
                NEXUS_LOGIN = 'nexuslogin'
		SONARSERVER = 'sonarserver'
        	SONARSCANNER = 'sonarscanner'
		}
	stages{
		stage('BUILD'){
			steps {
				sh 'mvn -s settings.xml -DskipTests install'
				}
			post {
				success {
					echo "Archiving"
					archiveArtifacts artifacts: '**/*.war'
						}
					}
				}
		stage('Test') {
			steps {
				sh 'mvn -s settings.xml test'
			}
		}
		stage('Checkstyle Analysis') {
			steps {
				sh 'mvn -s settings.xml checkstyle:checkstyle'
			}
		}
		stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }
		
	}
}
```

Trigger the build and see the result - The build has been succeeded 


![image](https://github.com/user-attachments/assets/4c615bb2-7e88-4fae-95f3-9f4cddeca70d)


If you check with SonarQube dashboard

![image](https://github.com/user-attachments/assets/ed72513c-3680-41af-adf6-ff5b995d3751)


## To add the SonarQuality Gate with Own Rule

SonarQube dashboard > Quality Gates > Create > Name > vprofileQG > save

Add Condition > On Overall code 

Quality Gate fails when > Bugs is greater than 25 > Add condition


![image](https://github.com/user-attachments/assets/392dbd6a-52d6-4d53-a677-d436ae393a94)


To attach this rule to your project

SonarQube dashboard > Project > Project Settings > Quality Gate > Choose > vprofileQG

![image](https://github.com/user-attachments/assets/f7138ecc-2887-45fe-8802-6f2ff9b61bc2)


To add the webhooks

SonarQube dashboard > Project > Project Settings > Webhooks > Create

```
Name - jenkinswebhook
URL - http://172.31.2.153:8080/sonarqube-webhook  //jenkins-private-ip
```

![image](https://github.com/user-attachments/assets/c80bd6cc-251c-4ef5-852d-df0959149edd)


Github > Jenkinsfile

```
pipeline {
	agent any
	tools {
		jdk "OracleJDK8"
                maven "MAVEN3.9"
		}
	environment {
		SNAP_REPO = 'devops-snapshots'
                NEXUS_USER = 'admin'
                NEXUS_PASS = 'admin@23'
                RELEASE_REPO = 'devops-release'
                CENTRAL_REPO = 'devops-central'
                NEXUSIP = '172.31.9.243'
                NEXUSPORT = '8081'
                NEXUS_GRP_REPO = 'devops-group'
                NEXUS_LOGIN = 'nexuslogin'
		SONARSERVER = 'sonarserver'
        	SONARSCANNER = 'sonarscanner'
		}
	stages{
		stage('BUILD'){
			steps {
				sh 'mvn -s settings.xml -DskipTests install'
				}
			post {
				success {
					echo "Archiving"
					archiveArtifacts artifacts: '**/*.war'
						}
					}
				}
		stage('Test') {
			steps {
				sh 'mvn -s settings.xml test'
			}
		}
		stage('Checkstyle Analysis') {
			steps {
				sh 'mvn -s settings.xml checkstyle:checkstyle'
			}
		}
		stage('Sonar Analysis') {
			environment {
				scannerHome = tool "${SONARSCANNER}"
				}
			steps {
				withSonarQubeEnv("${SONARSERVER}") {
					sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
					}
				}
			}
		stage("Quality Gate") {
			steps {
				timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
					waitForQualityGate abortPipeline: true
					}
				}
			}
		
	}
}
```

My build has been failed due to to quality gate failure  (its expected one)


![image](https://github.com/user-attachments/assets/32557959-b9ea-48e0-8262-eae243a1adaf)


Lets increase the Quality Gate value as 100 and start the build


![image](https://github.com/user-attachments/assets/241e1be8-b175-4a99-8cc2-3bdfd3f57b01)


Now my build has been succeeded


![image](https://github.com/user-attachments/assets/08498781-7781-48a4-97fa-0b52649d50e6)


![image](https://github.com/user-attachments/assets/dc852804-1d7a-4524-b130-289e272b582b)


## To Publish Artifacts to the Nexus Repository

We have the artifacts in workspace (below path). We have to send this artifact with timestamp to the Nexus Repository - devops-release

![image](https://github.com/user-attachments/assets/3c1e6958-00f2-4783-86f2-4dbedb256839)


To enable Build TimeStamp in Jenkins

Jenkins > System > Build TimeStamp > Enable

```
Timezone - Etc/UTC
Pattern - yy-MM-dd_HH:mm
Apply > Save
```

Github > Jenkinsfile

```
pipeline {
	agent any
	tools {
		jdk "OracleJDK8"
                maven "MAVEN3.9"
		}
	environment {
		SNAP_REPO = 'devops-snapshots'
                NEXUS_USER = 'admin'
                NEXUS_PASS = 'admin@23'
                RELEASE_REPO = 'devops-release'
                CENTRAL_REPO = 'devops-central'
                NEXUSIP = '172.31.9.243'
                NEXUSPORT = '8081'
                NEXUS_GRP_REPO = 'devops-group'
                NEXUS_LOGIN = 'nexuslogin'
		SONARSERVER = 'sonarserver'
        	SONARSCANNER = 'sonarscanner'
		}
	stages{
		stage('BUILD'){
			steps {
				sh 'mvn -s settings.xml -DskipTests install'
				}
			post {
				success {
					echo "Archiving"
					archiveArtifacts artifacts: '**/*.war'
						}
					}
				}
		stage('Test') {
			steps {
				sh 'mvn -s settings.xml test'
			}
		}
		stage('Checkstyle Analysis') {
			steps {
				sh 'mvn -s settings.xml checkstyle:checkstyle'
			}
		}
		stage('Sonar Analysis') {
			environment {
				scannerHome = tool "${SONARSCANNER}"
				}
			steps {
				withSonarQubeEnv("${SONARSERVER}") {
					sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
					}
				}
			}
		stage("Quality Gate") {
			steps {
				timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
					waitForQualityGate abortPipeline: true
					}
				}
			}
		stage("UploadArtifact"){
			steps{
				nexusArtifactUploader(
                                nexusVersion: 'nexus3',
                                protocol: 'http',
                                nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                                groupId: 'QA',
                                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                                repository: "${RELEASE_REPO}",
                                credentialsId: "${NEXUS_LOGIN}",
                                artifacts: [
					[artifactId: 'vproapp',
					 classifier: '',
					 file: 'target/vprofile-v2.war',
					 type: 'war']
					]
					)
				}
			}
		
	}
}
```

To start the build and check the Nexus repository whether the artifacts has been stored with Timestamp


![image](https://github.com/user-attachments/assets/0ce60d78-6962-40d0-b3d7-938ba37eed55)


My build has been succeeded as well the artifacts stored in Nexus repository.


![image](https://github.com/user-attachments/assets/e3e8a911-000e-4ef7-9ee5-51d5914b8486)



## To add a Slack Notification Stage

First to create a new workspace using below link

https://slack.com/get-started#/landing

```
Workspace - vprofilecicd
Mail - add your mail
Project - devopscicd
Start with the limited free edition
```

![image](https://github.com/user-attachments/assets/4af606c4-09f4-43f1-9c43-c68e2b3aa776)


To create a new channel named "jenkinscicd"

Slack dashboard > Add channel > To create a new channel > name > jenkinscicd > Create


![image](https://github.com/user-attachments/assets/c759778a-ab10-4459-bbca-6aae25e669ad)


To add apps to the Slack - Search : Slack Market Place - Apps and Integration

Find - jenkinsci > Add to Slack > Choose your channel > jenkinscicd > Add Jenkins CI integration

To copy the integration token credential id in somewhere for now > Save settings

```
token id - pOGohpp3Wm0GK8i9Petp2vXL
```

To Create a credential in jenkins dashboard

Jenkins Dashboard > Credential > System > Global Credential > Kind > Secret text


```
Secret - token
ID - slacktoken
Create
```

To configure the slack in Jenkins

Jenkins > System > Slack

```
Workspace - vprofilecicd-j1a6033  //you can get this URL from yur workspace - copy/paste in Jenkins except .slack.com
Credential - slacktoken
Default channel - #jenkinscicd
Test connect > Success
Apply & Save
```


![image](https://github.com/user-attachments/assets/d31fdee0-47cd-4021-b07f-53595748ab82)



To Start the build and check the status in Slack Channel - Build has been succeeded

![image](https://github.com/user-attachments/assets/5ea109a5-4d97-48fc-86f1-ecbaebff760d)


If you check with Slack channel


![image](https://github.com/user-attachments/assets/add5517c-1ada-4281-8552-59d614beccad)


## To setup the Github repo for CICD

First clone the below repo and checkout the docker branch in your local machine


https://github.com/kohlidevops/vprofile-project.git

```
git clone https://github.com/kohlidevops/vprofile-project.git
git checkout docker
//Copy the Docker-files folder
git checkout ci-jenkins
//create a new branch called cicd-jenkins
git checkout cicd-jenkins
//Place the Docker-files folder in cicd-jenkins branch
//To create two folders in cicd-jenkins branch
mkdir StagePipeline ProdPipeline
cp Jenkinsfile StagePipeline/
cp Jenkinsfile ProdPipeline/
//remove the Jenkinsfile in cicd-jenkins
git rm Jenkinsfile
```

![image](https://github.com/user-attachments/assets/172684b4-4df1-43ab-9681-a4d88862a139)


Now push the code to Github repo

```
git add.
git commit -m "preparing cicd pipeline"
git push origin cicd-jenkins
```

![image](https://github.com/user-attachments/assets/80c40ded-2f4e-4170-a654-cf4a7e5c61cb)


## To update the Jenkins Ip in Github webhook

Github > Your project > Settings > Webhook > Add a new webhook

```
Payload URL - http://55.106.75.123:8080/github-webhook/
content type - application/json
Which events would you like to trigger this webhook - Just the push event. 
Add webhook
```

![image](https://github.com/user-attachments/assets/18a42574-cbb9-4747-8c92-c29018af3787)


## To create an IAM user with policy

To create an IAM user with below policies to grant jenkins to access the ECS and ECR in AWS and store the access key and secret key in somewhere


![image](https://github.com/user-attachments/assets/16745097-48fd-4bec-bb7e-99ae0ab091a1)


## To create an AWS Elastic Container Registry

AWS > ECR > Create a Private repository > Name > vprofileappimg >

![image](https://github.com/user-attachments/assets/5e9a6842-6b92-45f7-8a22-879c64e5408f)


## To install the plugins in Jenkins 

To install below plugins in Jenkins dashboard

Jenkins > Manage plugins > Available > Install plugins

```
Docker Pipeline
CloudBees Docker Build and Push
Amazon ECR
Pipeline: AWS Steps
```

![image](https://github.com/user-attachments/assets/afd3a0ca-b5b0-4e25-841e-182cfb384aee)


## To add the IAM credential in Jenkins

Jenkins > Manage Jenkins > Credential > System > Global credential

```
Kind - AWS Credential
ID - awscreds
Access key - 1111111111111111
Secret key - ****************
Create
```

![image](https://github.com/user-attachments/assets/3b3c056f-21b4-47e9-80c3-fd77fb6d1773)


## To install AWS CLI and Docker Engine in Jenkins Server

SSH to Jenkins Server


```
sudo -i
apt update && apt install awscli -y
# Add Docker's official GPG key:
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
usermod -aG docker jenkins
su - jenkins
docker images
exit
sudo systemctl restart jenkins.service
```


## To add the Docker Build and Push stage in Jenkinsfile

Github > vprofile-project > StagePipeline > Jenkinsfile

```
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
	agent any
	tools {
		jdk "OracleJDK8"
                maven "MAVEN3.9"
		}
	environment {
		SNAP_REPO = 'devops-snapshots'
                NEXUS_USER = 'admin'
                NEXUS_PASS = 'admin@23'
                RELEASE_REPO = 'devops-release'
                CENTRAL_REPO = 'devops-central'
                NEXUSIP = '172.31.9.243'
                NEXUSPORT = '8081'
                NEXUS_GRP_REPO = 'devops-group'
                NEXUS_LOGIN = 'nexuslogin'
		SONARSERVER = 'sonarserver'
        	SONARSCANNER = 'sonarscanner'
		registryCredential = 'ecr:ap-south-1:awscreds'
                appRegistry = '590183829524.dkr.ecr.ap-south-1.amazonaws.com/vprofileappimg'
                vprofileRegistry = "https://590183829524.dkr.ecr.ap-south-1.amazonaws.com"
		}
	stages{
		stage('BUILD'){
			steps {
				sh 'mvn -s settings.xml -DskipTests install'
				}
			post {
				success {
					echo "Archiving"
					archiveArtifacts artifacts: '**/*.war'
						}
					}
				}
		stage('Test') {
			steps {
				sh 'mvn -s settings.xml test'
			}
		}
		stage('Checkstyle Analysis') {
			steps {
				sh 'mvn -s settings.xml checkstyle:checkstyle'
			}
		}
		stage('Sonar Analysis') {
			environment {
				scannerHome = tool "${SONARSCANNER}"
				}
			steps {
				withSonarQubeEnv("${SONARSERVER}") {
					sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
					}
				}
			}
		stage("Quality Gate") {
			steps {
				timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
					waitForQualityGate abortPipeline: true
					}
				}
			}
		stage("UploadArtifact"){
			steps{
				nexusArtifactUploader(
                                nexusVersion: 'nexus3',
                                protocol: 'http',
                                nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                                groupId: 'QA',
                                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                                repository: "${RELEASE_REPO}",
                                credentialsId: "${NEXUS_LOGIN}",
                                artifacts: [
					[artifactId: 'vproapp',
					 classifier: '',
					 file: 'target/vprofile-v2.war',
					 type: 'war']
					]
					)
				}
			}
		 stage('Build App Image') {
			 steps {
				 script {
					 dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
					 }
				 }
			 }
		stage('Upload App Image') {
			steps{
				script {
					docker.withRegistry( vprofileRegistry, registryCredential ) {
						dockerImage.push("$BUILD_NUMBER")
						dockerImage.push('latest')
						}
					}
				}
			}
		}
	post {
		always {
			echo 'Slack Notifications.'
			slackSend channel: '#jenkinscicdchannel',
				color: COLOR_MAP[currentBuild.currentResult],
				message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
			}
		}
	}
```


## To create CICD Pipeline in Jenkins Dashboard

Jenkins > new item > name > vprofile-cicd-pipeline > Pipeline > create

```
choose > GitHub hook trigger for GITScm polling
Pipeline > Definition > Pipeline Script from SCM
SCM - Git
Repository URL - git@github.com:kohlidevops/vprofile-project.git
Credentials - git
Branch Specifier - */cicd-jenkins
Script path - StagePipeline/Jenkinsfile
Apply and Save
```

To start the build and check the status - My build has been succedded


![image](https://github.com/user-attachments/assets/30cf5065-30c1-46c5-bace-5d7eac0ed78b)


If you check with your AWS ECR, you can see your images

![image](https://github.com/user-attachments/assets/718fe214-120a-4b7e-b390-00f435e02c88)


## To Setup Elastic Container Service in AWS for Staging

AWS > ECS > Create Cluster

```
Cluster name - vprostaging
Namespace - vprostaging
Infrastructure - Fargate
Monitoring > Enable > Container Insights
Tags > Name > vprostaging
Create a Cluster
```

![image](https://github.com/user-attachments/assets/fcd7729e-193c-42a7-ab0a-45fc6f420b30)


#### To create a Task definition

AWS > ECS > Task definition

```
Task definition family > vproappstagetask
Infrastructure requirements > Launch type > AWS Fargate
Operatin system > Linux/X86_64
CPU > 1
Memory > 3
Task role > None
Task execution role > ECSTaskExecutionRole
Container details > Name > vproapp
Image URI > 123456789.dkr.ecr.ap-south-1.amazonaws.com/vprofileappimg  //your ECR Image URI
Port mapping > Container port > 8080
//Rest of things leave as default and create a Task definition
```

![image](https://github.com/user-attachments/assets/f0d59f45-dcc5-4790-83ff-805be911fe58)


#### To create a Service in ECS

AWS > ECS > Cluster > Choose your Cluster > Create a Service

```
Computer configuration > Launch type > Fargate
Platform version > Latest
Deployment configuration > Application type > Service
Task definition > Choose > Specify revision manually
Family > vproappstagetask
Service name > vproappstagesvc
Service type > Replica
Desired task > 1
Networking > VPC > Select your VPC
Subnets > Select all subnets
Security group > New > vprappstage-sg
Inbound rule > 8080 with Anywhere
Loadbalancing > Use Loadbalancing
Load balancer type > ALB
container > vproapp 8080:8080
Create a new Loadbalancer > vproappstage-alb
Create a new listener > 80 with HTTP
Create a new target group > vproappstag-tg
Protocol > HTTP
Deregistration delay > 120
Healthcheck protocol > HTTP
Healthcheck path > /login
Create

Edit your Target group > override > Healthcheck port > 8080 > save changes
```

![image](https://github.com/user-attachments/assets/e91051b7-65a8-44bd-aa08-fe3c72e54bc9)


New Loadbalancer has been created and it mapped the target group as we have configured

![image](https://github.com/user-attachments/assets/03b625c6-4ede-40e7-af6d-5fbd84fd57db)


If you hit the ALB URL in browser, then our container has been running successfully in ECS


![image](https://github.com/user-attachments/assets/e5aa2336-1141-4b76-b905-fb7f9dc9ba1f)


#### To update the stage in StagePipeline Jenkinsfile

Github > vprofile-project > (branch - cicd-pipeline) > StagePipeline > Jenkinsfile > Edit and update

```
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
	agent any
	tools {
		jdk "OracleJDK8"
                maven "MAVEN3.9"
		}
	environment {
		SNAP_REPO = 'devops-snapshots'
                NEXUS_USER = 'admin'
                NEXUS_PASS = 'admin@23'
                RELEASE_REPO = 'devops-release'
                CENTRAL_REPO = 'devops-central'
                NEXUSIP = '172.31.9.243'
                NEXUSPORT = '8081'
                NEXUS_GRP_REPO = 'devops-group'
                NEXUS_LOGIN = 'nexuslogin'
		SONARSERVER = 'sonarserver'
        	SONARSCANNER = 'sonarscanner'
		registryCredential = 'ecr:ap-south-1:awscreds'
                appRegistry = '590183829524.dkr.ecr.ap-south-1.amazonaws.com/vprofileappimg'
                vprofileRegistry = "https://590183829524.dkr.ecr.ap-south-1.amazonaws.com"
		cluster = "vprostaging"
		service = "vproappstagesvc"
		}
	stages{
		stage('BUILD'){
			steps {
				sh 'mvn -s settings.xml -DskipTests install'
				}
			post {
				success {
					echo "Archiving"
					archiveArtifacts artifacts: '**/*.war'
						}
					}
				}
		stage('Test') {
			steps {
				sh 'mvn -s settings.xml test'
			}
		}
		stage('Checkstyle Analysis') {
			steps {
				sh 'mvn -s settings.xml checkstyle:checkstyle'
			}
		}
		stage('Sonar Analysis') {
			environment {
				scannerHome = tool "${SONARSCANNER}"
				}
			steps {
				withSonarQubeEnv("${SONARSERVER}") {
					sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
					}
				}
			}
		stage("Quality Gate") {
			steps {
				timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
					waitForQualityGate abortPipeline: true
					}
				}
			}
		stage("UploadArtifact"){
			steps{
				nexusArtifactUploader(
                                nexusVersion: 'nexus3',
                                protocol: 'http',
                                nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                                groupId: 'QA',
                                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                                repository: "${RELEASE_REPO}",
                                credentialsId: "${NEXUS_LOGIN}",
                                artifacts: [
					[artifactId: 'vproapp',
					 classifier: '',
					 file: 'target/vprofile-v2.war',
					 type: 'war']
					]
					)
				}
			}
		 stage('Build App Image') {
			 steps {
				 script {
					 dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
					 }
				 }
			 }
		stage('Upload App Image') {
			steps{
				script {
					docker.withRegistry( vprofileRegistry, registryCredential ) {
						dockerImage.push("$BUILD_NUMBER")
						dockerImage.push('latest')
						}
					}
				}
			}
		stage('Deploy to ECS staging') {
			steps {
				withAWS(credentials: 'awscreds', region: 'ap-south-1') {
					sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
					}
				}
			}
		}
	post {
		always {
			echo 'Slack Notifications.'
			slackSend channel: '#jenkinscicdchannel',
				color: COLOR_MAP[currentBuild.currentResult],
				message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
			}
		}
	}
```
 

Once updated the code, then automatically build has been started in Jenkins  - The build has been succeeded and new task is assigned

![image](https://github.com/user-attachments/assets/19b5b589-5cdc-4401-8a75-0dcfb0a0afbb)


New task is assigned just now


![image](https://github.com/user-attachments/assets/7ea7f524-e83b-41c2-93b0-b659dbe02273)



## To Setup Elastic Container Service in AWS for Production

AWS > ECS > Create Cluster

```
Cluster name - vproprod
Namespace - vproprod
Infrastructure - Fargate
Monitoring > Enable > Container Insights
Tags > Name > vproprod
Create a Cluster
```

![image](https://github.com/user-attachments/assets/5880f2ea-38ec-4b16-b5b4-a943521d5040)


#### To create a Task definition

AWS > ECS > Task definition

```
Task definition family > vproappprodtask
Infrastructure requirements > Launch type > AWS Fargate
Operatin system > Linux/X86_64
CPU > 1
Memory > 3
Task role > None
Task execution role > ECSTaskExecutionRole
Container details > Name > vproapp
Image URI > 123456789.dkr.ecr.ap-south-1.amazonaws.com/vprofileappimg  //your ECR Image URI
Port mapping > Container port > 8080
//Rest of things leave as default and create a Task definition
```

![image](https://github.com/user-attachments/assets/5f8298c2-6846-4d1a-9bc9-2408604387ff)


#### To create a Service in ECS

AWS > ECS > Cluster > Choose your Cluster > Create a Service

```
Computer configuration > Launch type > Fargate
Platform version > Latest
Deployment configuration > Application type > Service
Task definition > Choose > Specify revision manually
Family > vproappprodtask
Service name > vproappprodsvc
Service type > Replica
Desired task > 1
Networking > VPC > Select your VPC
Subnets > Select all subnets
Security group > New > vprappprod-sg
Inbound rule > 8080 with Anywhere
Loadbalancing > Use Loadbalancing
Load balancer type > ALB
container > vproapp 8080:8080
Create a new Loadbalancer > vproappprod-alb
Create a new listener > 80 with HTTP
Create a new target group > vproappprod-tg
Protocol > HTTP
Deregistration delay > 120
Healthcheck protocol > HTTP
Healthcheck path > /login
Create

Edit your Target group > override > Healthcheck port > 8080 > save changes
```

New service has been created for prodsvc

![image](https://github.com/user-attachments/assets/4d1967ec-7ead-4af2-a9d1-861010ac3e80)


The prod loadbalancer has been created

![image](https://github.com/user-attachments/assets/b0f05ca0-418f-4223-9afb-27f1cda1b720)


If you access the ALB URL

![image](https://github.com/user-attachments/assets/52cecf44-d9c8-4277-91ad-9afe318f9f90)


#### To create a new branch for Prod

Go to your local machine > your project > create a new branch called prod from cicd-jenkins branch

```
git checkout -b prod
ls
git add .
git commit -m "new branch"
git push origin prod
```

![image](https://github.com/user-attachments/assets/03281e64-9f62-42bb-8f74-79f0087dc5e2)


Check your github repository


![image](https://github.com/user-attachments/assets/11ec9956-20c1-4ccb-a9e2-01224ddbaf7f)


Github repo > vprofile-project > prod branch > edit > Jenkinsfile > commit the code

```
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
	agent any
	tools {
		jdk "OracleJDK8"
                maven "MAVEN3.9"
		}
	environment {
		cluster = "vproprod"
		service = "vproappprodsvc"
		}
	stages{
		stage('Deploy to ECS Production') {
			steps {
				withAWS(credentials: 'awscreds', region: 'ap-south-1') {
					sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
					}
				}
			}
		}
	post {
		always {
			echo 'Slack Notifications.'
			slackSend channel: '#jenkinscicdchannel',
				color: COLOR_MAP[currentBuild.currentResult],
				message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
			}
		}
	}
```


#### To create a new CICD Pipeline for Production in Jenins

Jenkins > New Item > Name > vpro-cicd-prod-pipeline > Copy from > vprofile-cicd-pipeline > Create

```
Choose > Github hook trigger for GITScm polling
Pipeline > Definition > Pipeline Script from SCM
SCM > Git
Repository URL > git@github.com:kohlidevops/vprofile-project.git
Credentials > git //your token
Branch Specifier > */prod
Script Path > ProdPipeline/Jenkinsfile
Apply & Save
```

Start the Build > Build has been succeeded


![image](https://github.com/user-attachments/assets/7fe6fcff-29d6-4619-b3b5-584a9a15abb6)


If you check with the Prod ECS Task the new task has been created


![image](https://github.com/user-attachments/assets/fc1d86ea-6939-4f1f-bdad-e472da14de67)


#### To promote the deployment from Staging to Prod once Approved

Go to your Local machine > vprofile-project > cicd-jenkins branch (staging branch) > do some change in README.md > commit

```
git checkout cicd-jenkins
git add .
git commit -m "new update"
git push origin cicd-jenkins
```

Now the Staging Pipeline should be triggered and deploy the image on Staging ECS

![image](https://github.com/user-attachments/assets/29ec6131-5b53-4f50-abf2-c599d2bbed87)


Once the Staging App is approved, then promote this to Prod Pipeline

![image](https://github.com/user-attachments/assets/8c50579e-1ee6-476d-9209-226c84b6e063)


Navigate to your repo > then do below activity


```
git checkout prod
git merge cicd-jenkins
git push origin prod
```

Now the Prod Pipeline should be triggered and deploy the image on Prod ECS

![image](https://github.com/user-attachments/assets/a24c717c-7289-46a8-af1a-759b19487aea)


![image](https://github.com/user-attachments/assets/5f2a5d86-aa03-4cc0-b3ea-ecfbcc338b3b)






