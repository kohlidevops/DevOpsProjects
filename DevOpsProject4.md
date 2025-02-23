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


5. To Setup and Configure Sonar Cloud

Login > https://sonarcloud.io/login > with GitHub > Profile Settings > My Account > Security > name it - vpro-sonar-cloud > Generate token

```
token - 7c28266f5c2662f9a4e090407f342cd0e00c93ee
```

Profile Settings > My Organization > Create a new Organization

```
Name - latchuprofile
Key - latchuprofile
Choose a Plan - Free
Create Organization
Select - Analyze a new project
Organization - latchuprofile
Display Name - latchuprofile-repo
Project Key - latchuprofile-repo
Project visibility - Public > Next > Previous Version > Create Project
```

If you click Information, then you can find all the configuration details

![image](https://github.com/user-attachments/assets/6bcab4c6-43e1-439f-9c38-a29a892f91df)


6. To Setup a AWS SSM Parameter store to store values

AWS > SSM > Paramter Store > Create Paramter

To create a parameter for Organization

```
Name - Organization
Tier - Standard
Type - String
Data Type - text
Value - latchuprofile  ##Organization Key in SonarCloud
Create
```

To create a parameter for HOST

```
Name - HOST
Tier - Standard
Type - String
Data Type - text
Value - https://sonarcloud.io
Create
```

To create a parameter for Project


```
Name - Project
Tier - Standard
Type - String
Data Type - text
Value - latchuprofile-repo  ##Project key in sonarcloud
Create
```

To create a parameter for LOGIN

```
Name - LOGIN
Tier - Standard
Type - SecureString
Data Type - text
KMS Key source - My Current Account
Value - 7c28266f5c2662f9a4e090407f342cd0e00c93ee  ##Token created in sonarcloud
Create
```

![image](https://github.com/user-attachments/assets/fbd97f70-2767-451d-836c-2ed33b9e063a)


7. To Setup AWS Code Build for SonarQube Code Analysis

AWS Code Artifact > Repositories > maven-central-store > view connection instruction

```
Operating system - Mac & Linux
Package manager client - mvn
Configuration method - pull from your repository
#You can get the code artifact url - Copy and replace this url in pom.xml in bitbucket cloud (end of the section)
```

![image](https://github.com/user-attachments/assets/9f24a42f-1638-44a4-ab8c-16e7749319ee)


Place the URL in profile section & mirrors section in settings.xml file

![image](https://github.com/user-attachments/assets/9c4b5078-dbc6-472e-851e-5a1166bc571a)


![image](https://github.com/user-attachments/assets/86fc67c2-82f6-4de4-bd5a-023d154678f8)


To add the buildspec.yml in root directory of the project source code in bitbucket cloud

```
https://github.com/kohlidevops/DevOpsProjects/blob/main/aws-ci-buildspec.yml
```

![image](https://github.com/user-attachments/assets/418360fd-d01f-462e-ad04-235955bebabf)


To update authorization token in buildspec.yml

You can collect the token from code artifact > view connection metrics

![image](https://github.com/user-attachments/assets/7ac8a3e3-b832-48eb-b6ac-8470264b2a8b)


To create a Code Build 

AWS > Code Build > Create a Project

```
Project name - vpro-code-analysis
Project type - Default Project
Source > Primary - Bitbucket
#Choose Manage credential
Credentail Type - OAuth app
Service - Code Build > Connect to bitbucket
```

![image](https://github.com/user-attachments/assets/50e692da-4212-43fc-b59c-da64cbbdf42d)

Once access is granted, you can see your repository in AWS

![image](https://github.com/user-attachments/assets/2d1cc022-e5f9-44f4-b72d-446c7e6fd899)


```
Source version - ci-aws
Environment > Provisioning model - On-demand
Environment Image - Managed image
Compute - EC2
Operating system - Ubuntu
Runtime - Standard
Image - aws/codebuild/standard:7.0
Image version - always use the latest version for this runtime version
Service Role > New Role > Role name - codebuild-vpro-code-analysis-service-role
Buildspec > Build specification - Use a buildspec file
Cloudwatch logs > Enable (If need)
Create a build project
```

To create an IAM Policy for SSM

AWS > IAM > Policy > Create > Visual

```
Service - System Manager
Actions > List - Describe Parameters
Read - DescribeDocumentParameters, GetParameterHistory, GetParameters, GetParameter, and GetParametersByPath
Policy name - vpro-ssm-policy
create
```

To associate the IAM policy to Code Build IAM Role

You can check the IAM role from Code Build - Then associate the Policy to IAM role - Because Code Build needs to access the SSM Parameter store service to run and add CodeArtifactRealdOnlyAccess.

![image](https://github.com/user-attachments/assets/d5431fee-f591-4cf4-997a-bcc656a1e599)


To Start the Build

AWS > Code Build > Your Project > Start the Build

![image](https://github.com/user-attachments/assets/d60cde2b-e47f-43a0-868e-28cd322074a6)

The build has been succeeded

![image](https://github.com/user-attachments/assets/d1127a3a-d8f6-4d8c-bde6-5776c456b01a)


If you check with sonarcloud dashboard

![image](https://github.com/user-attachments/assets/70557fcc-aae1-4f68-a23d-4bbe7258bc1c)


```
Code Build fetch the code and do the code analysis
Upload the results to Sonar Cloud
Download the dependencies from the Artifacts whenever required
The next Build project will build the Artifacts and store it to the S3 bucket
```

To update the buildspec yaml for build the proect

```
https://github.com/kohlidevops/DevOpsProjects/blob/main/aws-ci-buildspec.yaml
```

To update the Authorization token in buildspec yaml (bitbucket repo)

![image](https://github.com/user-attachments/assets/e7864e67-d7b4-4b7a-a517-af826b7a4d0c)


8. To create a Code Build Project for Build Artifacts

AWS > Code Build > Create a new project

```
Project name - vprofile-build-artifacts
Project type - Default project
Source > Primary > Source provider - Bitbucket
Repository - Reposiory in my Bitbucket account - https://bitbucket.org/latchudevops/vprofile-project.git
Source version - ci-aws
Provisioning model - On-demand
Environment image - Managed Image
Compute - EC2
Operating system - Ubuntu
Runtime - Standard
Image - aws/codebuild/standard:7.0
Service Role > New > Role name > codebuild-vprofile-build-artifacts-service-role
Build spec > Use a Build spec file
Buildspec name - aws-files/build_buildspec.yml
CloudWatch logs - Enable (if need)
Create a build project
```

To add a Policy to an IAM Role

AWS > IAM > Roles > Choose the Build-Artifact Role > Add permissions > Add CodeArtifactsReadOnlyAccess

![image](https://github.com/user-attachments/assets/377b4df9-a728-4b02-ada1-10db41076666)


To Start the build

AWS > Code Build > vprofile-build-artifacts > Start Build


My build has been succeeded

![image](https://github.com/user-attachments/assets/e8a45b5e-77a0-4be6-822d-3562541a9c97)


9. To Setup a Code Pipeline




