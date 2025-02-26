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


9. To Create a S3 Bucket

Create a S3 bucket and named it - vpro-build-artifacts

To create a folder called pipeline-artifacts

![image](https://github.com/user-attachments/assets/a43ff343-4a93-449d-96c0-3f578ba8e049)


10. To create a SNS

To create a SNS Topic and subscribe with Email

![image](https://github.com/user-attachments/assets/8dc1f53e-b049-41b9-81d9-eff93798d2cb)


11. To Create a Code Pipeline

AWS > Code Pipeline > Build with Custom Pipeline

![image](https://github.com/user-attachments/assets/3f120198-19f2-49dd-9816-f7a84b59c6be)


```
Pipeline name - vpro-ci-pipeline
Execution mode - Queued
Service role > New Service Role > AWSCodePipelineServiceRole-ap-south-1-vpro-ci-pipeline
Source provider - Bitbucket > Connect to Bitbucket (Create a new connection with new app)
Repository name - latchudevops/vprofile-project
Default branch - ci-aws
Add build stage > Build Provider > Other Build Provider > AWS Code Build
Project name - vprofile-build-artifacts
Build type - single
Input artifacts - SourceArtifact
Skip - Test stage and deploy stage > Then create a Pipeline
```

The Pipeline will execute automatically once it is created - So you can stop it as of now

12. To add the CodeAnalysis stage

Choose > Pipeline > Edit > Add the stage after Code Commit stage > Name it - CodeAnalysis > Add Action group 

```
Action name - SonarCodeAnalysis
Action provider - AWS CodeBuild
Region - Mumbai
Input artifacts - SourceArtifact
Project name - vpro-code-analysis
Build type - Single
Done
```


![image](https://github.com/user-attachments/assets/7b13ee27-3980-4cdc-98c4-2cb776f4d786)


13. To add the Deploy stage

To add one more stage after Build stage > name it > Deploy > Add action group > DeployToS3Bucket

```
Action name - DeployToS3Bucket
Action provider - Amazon S3
Region - Mumbai
Input artifacts - BuildArtifact
Bucket - latchu-build-artifacts
Deployment path - pipeline-artifacts  #folder-name
Choose > Extract file before deploy
Done
```

Finally Save the Code Pipeline


14. To Setup a Notification

Choose your Pipeline > Settings > Notification > Create a Notification rule

```
Notification name - vpro-notification
Detail type - Full
Events that trigger notifications - Select All
Targets > SNS Topic - choose your topic
Submit
```

![image](https://github.com/user-attachments/assets/469fe2db-267b-4e13-83f1-65e14e32969e)


15. To trigger the Pipeline manually

Choose your Pipeline > Release change > to trigger manually

My CI Pipeline has been succeeded


![image](https://github.com/user-attachments/assets/9753d62d-434a-40f3-ab4e-192cbfb191b8)


The buil artifacts (war file) to deploy on S3 bucket



![image](https://github.com/user-attachments/assets/41ec556b-569e-4810-8464-6922af6072b6)



Try to do some changes in Bitbucket repo and check the Pipeline whether it is trigerred automatically or not


![image](https://github.com/user-attachments/assets/eeab9edb-bf8f-4ab4-8485-f7ffd47bae22)


This is the CI stages


![image](https://github.com/user-attachments/assets/aab45b4e-1bfd-4dad-af38-bc8f65d99c39)


**To delete the CI Pipeline and we will create new Pipeline with CICD process**


16. To Create an IAM Role for Elastic Beanstalk EC2 Instance Profile

IAM Role > Create new Role > Trusted entity type > AWS Service > Use case > EC2


![image](https://github.com/user-attachments/assets/b06e77e3-da8e-42f9-9f16-5d07608a2988)


17. To create an Elastic Beanstalk with Webserver Environment

AWS > Elastic Beanstalk


```
Environment tier - Web Server environment
Application name - vpro
Platform type - Managed Platform
Platform - Tomcat
Platform branch - Tomcat 10 with Corretto 21 running on 64 bit Amazon Linux 2023
Platform version - 5.4.3
Applicatio code - Sample application
Presets - custom configuration
Service role - Create and use new service role
EC2 key pair - your key pair
EC2 instance profile - Select your EC2 instance profile role
VPC and subnets - Select your own
Public IP address - Activated
//Leave the database subnets
Root volume - default
EC2 security group - your SG
Auto scaling group > Environment - Loadbalanced
Min - 2 and max - 4
Instance type - t3.micro
AMI ID - default
Loadbalancer subnets - All
Loadbalancer type - Application Loadbalancer with Dedicated
Processes > Default > Edit > Sessions > Enable - Session stickiness > save
Deployment policy - Rolling
Deployment bacth size - 50
Review and Create an Elastic Beanstalk
```

![image](https://github.com/user-attachments/assets/5e21790f-c2f4-47e8-93b9-07c4fddd72c7)



18. To Setup MYSQL RDS Instance

AWS > RDS > Create a new RDS

```
Engine option - MySQL
Engine version - MySQL 8.0.35
Template - Free tier
DB instance identifier - vpro-rds
Master username - admin
Credential management - Self managed
Master password - *****
Instance family - db.t4g.micro
Storage > GP3 > 20 GB
VPC - Select your own
Security gorup - RDS-SG
Initial database name - accounts
Create database
```

![image](https://github.com/user-attachments/assets/91026051-c07c-45b5-98c1-5b40a28224a2)


19. To Import the DB to MYSQL RDS

SSH to any one of the EC2 instance and execute below things
```
sudo -i
dnf install mariadb105 -y
//To download db file
wget https://raw.githubusercontent.com/kohlidevops/vprofile-project/refs/heads/cd-aws/src/main/resources/db_backup.sql
mysql -h <endpoint> -u username -p <db-name> < db_backup.sql
//enter your password
mysql -h <endpoint> -u username -p <db-name>
//enter your password
show tables;
exit
```

![image](https://github.com/user-attachments/assets/3f4f7718-83b2-4caa-9731-f83e4b502780)


20. To update the code with POM and settings XML files

AWS > CodeArtifact - To copy the Artifact URL from maven-central-store - view connection string

Bitbucket > branch > cd-aws > Edit > pom.xml and update the URL

![image](https://github.com/user-attachments/assets/d2aa721e-d262-4e50-af6d-7f47cdc328e1)


Bitbucket > branch > cd-aws > Edit > settings.xml and update the URL

![image](https://github.com/user-attachments/assets/8928eec8-bc45-46e4-926c-ef98189906bd)


21. To update all build spec yaml files

Bitbucket > your project > branch > cd-aws > choose > aws-files

Edit > sonar_buildspec.yml > To add the export authorization token (which is copied from maven central store instruction in aws code artifact) and comment "#CODEARTIFACT_AUTH_TOKEN: CODEARTIFACT_AUTH_TOKEN"

![image](https://github.com/user-attachments/assets/2bd01572-f410-4dc0-9aed-0d6ac26609d4)


Edit > build_buildspec.yml > To add the export authorization token (which is copied from maven central store instruction in aws code artifact) and comment "#CODEARTIFACT_AUTH_TOKEN: CODEARTIFACT_AUTH_TOKEN"

Edit > buildAndRelease_buildspec.yml > To add the export authorization token (which is copied from maven central store instruction in aws code artifact) and comment "#CODEARTIFACT_AUTH_TOKEN: CODEARTIFACT_AUTH_TOKEN"

22. To update the branch in Code Build Projects

To update the Source version as cd-aws in both vpro-code-analysis and vprofile-build-artifacts

![image](https://github.com/user-attachments/assets/9523b448-7701-41bd-8dbd-d3b0f28a4f5f)


23. To update the buildspec yaml to vpro-code-analysis build

Edit > vpro-code-analysis > Buildspec > Buildspec name - aws-files/sonar_buildspec.yml > save

![image](https://github.com/user-attachments/assets/d9b24788-b2f9-4163-9f56-e802ea7673ae)


24. To create a Parameter store for RDS

AWS > SSM > Parameter Store > Add

```
RDS-Endpoint - <endpoint>
RDSUSER - admin
RDSPASS - *****
```

![image](https://github.com/user-attachments/assets/ff23c25b-b07c-413e-826f-f53ebfbdc6b7)


25. To create a CodeBuild for Build and Release

AWS > CodeBuild > Create a new project

```
Project name - vpro-BuildAndRelease
Project type - Default
Source provider - Bitbucket
Repository > Repository in my Bitbucket account > choose your repo
Source version - cd-aws
Operating system - Ubuntu
Service role - Existing service role (codebuild-vpro-code-analysis-service-role)
Buildspec > use a buildspec file - aws-files/buildAndRelease_buildspec.yml
Cloudwatch logs - enable (if need)
Create build project
```

To start the build and know the status

![image](https://github.com/user-attachments/assets/ef7a4a1b-95ff-4fb5-9122-271ecb35a2f2)


26. To create a CodeBuild for Selenium Test

AWS > CodeBuild > Create a new project

```
Project name - SoftwareTesting
Project type - Default
Source provider - Bitbucket
Repository > Repository in my Bitbucket account > choose your repo
Source version - seleniumAutoScripts
Operating system - Windows Server 2019
Runtime - Base
Image windows-base:2019-3.0
Service role - Existing service role (codebuild-vpro-code-analysis-service-role)
Buildspec > Insert build commands - Editor
//Copy paste the below content and update the beanstalk URL
//https://github.com/kohlidevops/DevOpsProjects/blob/main/win_buildspec.yml
Artifacts > Primary > S3
Bucket name - latchu-build-artifacts
Name - Testcases //folder - Create if not available
Cloudwatch logs - enable (if need)
Create build project
```



