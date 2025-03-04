# Continuous Integration Using Jenkins, Nexus, SonarQube and Slack

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

