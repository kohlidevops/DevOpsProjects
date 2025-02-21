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
Name - JDK17
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
		jdk "JDK17"
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
