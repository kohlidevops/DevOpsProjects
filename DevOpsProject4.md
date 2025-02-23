# Continuous Integration (CI) on AWS CLoud using Atlassian Bitbucket, Code Artifact, SSM Parameter Store, Sonar Cloud, Code Build, SNS and Code Pipeline

1. To Create a Bitbucket repository in Atlassian Bitbucket Cloud


![image](https://github.com/user-attachments/assets/5f500d09-c26c-4f8a-ae37-bc402c89df98)


2. To integrate local and Bitbucket

Local machine > Go to C:/Users/latchu/.ssh/

```
ssh-keygen -t rsa
filename - bitbucket
ls -lh
```

Copy / Paste the bitbucket.pub to Bitbucket Cloud

Bitbucket repo > Click on gear or cog symbol > Personal bitbucket settings > SSH Keys > Add SSH key > Name it and add the bitbucket.pub key > save it

Go to - Local machine > Go to C:/Users/latchu/.ssh/

```
vim config

#bitbucket.org
Host bitbucket.org
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/bitbucket

ssh -T git@bitbucket.org
#Now it should authenticated
```

3. To clone the repo to your local machine

Go to - D:\DevOpsProjects

```
cd D:\DevOpsProjects
mkdir awscicd
cd awscicd
git clone https://github.com/kohlidevops/vprofile-project.git
ls -lh
cd vprofile-project
for i in `git branch -a | grep remotes | grep -v HEAD | cut -d / -f3`; do git checkout $i;done
git remote rm origin

#Go to Bitbucket repo - copy the git URL as SSH

git remote add origin git@bitbucket.org:latchudevops/vprofile-project.git
git push origin --all
```

![image](https://github.com/user-attachments/assets/91489a15-d421-4273-8f17-8214dc1a5136)


4. To create a repository in AWS Code Artifacts

AWS > Code Artifact > Create Repository

```
Repository name - vprofile-maven-repo
Public upstream repositories - maven-central-store
AWS account - This account
Domain name - latchudevops
Create
```

Two repository has been created

![image](https://github.com/user-attachments/assets/16b46882-0bce-4ef4-9a18-85f6df9df1bb)


Choose > maven-central-store repo > View connection instruction > 

```
operating system - Mac and Linux
Choose Package Manager client - mvn
Select a configuration method - pull from repository
#Now you can see the authorization token, server, profile and mirror
#Go through the sonar-buildspec yaml and settings xml
#project - https://github.com/kohlidevops/vprofile-project
#branch - ci-aws
```

 





