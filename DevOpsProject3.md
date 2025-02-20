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


