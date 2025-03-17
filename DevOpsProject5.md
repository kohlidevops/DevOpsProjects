# To deploy application in ECS Cluster using GitHub Actions CICD Workdlows, Maven, SonarQube, QualityGate Status, Docker, Amazon ECR and Amazon ECS

1. To setup the GitHub repo

2. To setup SSH login and integrate with GitHub repo

3. To test the code with Maven, CheckStyle and analysis the code with SonarQube

4. To upload the result to Sonar Cloud to check the Quality status

5.  To build the docker image and push to the Amazon Elastic Container Registry

6.  To deploy the docker image to the ECS Cluter

7.  To setup RDS for Application container to access the database

8.  Users can access the application using Loadbalancer which is created by ECS task


## To Setup GitHub Repo

Just fork below repository wihtout choosing "main" branch

```
https://github.com/kohlidevops/hprofile
```

## To Setup SSH login and integrate with GitHub repo

Login to your local machine 

```
cd Users/lenovoS340/.ssh/
ssh-keygen.exe
Enter file in which to save the key (/c/Users/lenovoS340/.ssh/id_rsa): githubaction
//now githubaction and githubaction.pub keys are available
cat githubaction.pub
//copy the content of public key and store it in the Github repo
```

Navigate to GitHub repo 

Profile > Settings > SSH and GPG Keys > New SSH key > Title - githubaction > key type - Authentication key > Add > Public key > Add SSH Key

```
cd D:\DevOpsProjects
ssh -i ~/.ssh/githubaction -T git@github.com
export GIT_SSH_COMMAND="~/.ssh/githubaction"
mkdir.exe githubaction
cd githubaction
git clone git@github.com:kohlidevops/hprofile.git
else
git clone https://github.com/kohlidevops/hprofile.git
```

![image](https://github.com/user-attachments/assets/423aaf08-782c-4b71-b329-253f55022ed1)


![image](https://github.com/user-attachments/assets/48ebfddc-e253-4730-ae9b-32d71510954b)

To update the README.md and push the code to GitHub repo

```
cd hprofile
vim README.md
//add some things
git add .
git commit -m "updated"
git push
```

## To Test your workflow

Go to your repo in GitHub > Goto Actions > Create by yourself > main.yml

```
name: Hprofile Actions
on: workflow_dispatch
jobs:
  Testing:
    runs-on: ubuntu-latest
    steps:
      - name: Testing Workflow
        run: echo "Workflow working perfectly"
```

Commit the changes > Choose > Actions > Hprofile Action > Run Workflow > Branch: Main > Run Workflow


![image](https://github.com/user-attachments/assets/e944c6ec-3e8b-403f-b7bc-f8f10f7331d3)


Now Build has been succeeded

![image](https://github.com/user-attachments/assets/2c86ad46-81b1-40b7-b840-51326587e53a)


## To add the Git Checkout, Maven Test and CheckStyle stage

Refer - https://github.com/marketplace/actions/checkout

Open your main.yml and update the code

```
name: Hprofile Actions
on: workflow_dispatch
jobs:
  Testing:
    runs-on: ubuntu-latest
    steps:
      - name: GitHub Code checkout
        uses: actions/checkout@v4

      - name: Maven Test
        run: mvn test

      - name: Maven Checkstyle
        run: mvn checkstyle:checkstyle
```

To run the build using Run Workflow - Thats working fine


![image](https://github.com/user-attachments/assets/3aed4c70-8a50-4f6e-a76a-e75a6d4ce49b)


## To add the SonarCode Analysis Stage

To create a SonarCloud and signin with GitHub account using below link


https://sonarcloud.io/projects


![image](https://github.com/user-attachments/assets/32015be5-8212-414e-a3d1-cb8b19257811)


Here we have to create a Organization, Project in Organization and login token and all the information should stored in GitHub secrets

To create a new Organization using "+" symbol near your profile in sonarcloud


![image](https://github.com/user-attachments/assets/2903479c-d76b-4352-9bb8-12b8696a66b4)

```
Create > Manually
Name > hprofile
Key > hprofilelatchu21
Plan > Free
Create > Organization
Analyze a new Project
Organization > hprofile
Display name > latchuactionscode
Projectkey > latchuactionscode
Project visibility > Public > Next > Previous version > Create Project
```

![image](https://github.com/user-attachments/assets/846416a4-3a29-4bc5-a015-80b5c93fde7f)


To generate the Token to grant access to GitHub Actions to authenticate

My Account > Security > Generate Token > Name it > githubactions > generate token

```
token - 111ae2069920a209c338235e7d11e6bddd13ff97
```

To store the secrets in GitHub repo

Go to your project > Settings > Secrets and Variables > Actions > Secrets > New secrets

```
SONAR_URL - https://sonarcloud.io
SONAR_TOKEN - 111ae2069920a209c338235e7d11e6bddd13ff97
SONAR_ORGANIZATION - hprofilelatchu21 //Your Key not organization name
SONAR_PROJECT_KEY - latchuactionscode
```


![image](https://github.com/user-attachments/assets/8fe63c52-bafe-45c1-a933-b0eac41f824b)


You can refer below link for sonarscanner workflow

https://github.com/marketplace/actions/sonar-scanner


To update the sonarscanner stage in GitHub Action main.yml

```
name: Hprofile Actions
on: workflow_dispatch
jobs:
  Testing:
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Maven test
        run: mvn test

      - name: Maven Checkstyle
        run: mvn checkstyle:checkstyle

      # Setup java 11 to be default (sonar-scanner requirement as of 5.x)
      - name: Set Java 11
        uses: actions/setup-java@v3
        with:
         distribution: 'temurin' # See 'Supported distributions' for available options
         java-version: '11'

    # Setup sonar-scanner
      - name: To Setup SonarQube
        uses: warchant/setup-sonar-scanner@v7
   
    # Run sonar-scanner
      - name: To Run SonarQube Scan
        run: sonar-scanner
           -Dsonar.host.url=${{ secrets.SONAR_URL }}
           -Dsonar.login=${{ secrets.SONAR_TOKEN }}
           -Dsonar.organization=${{ secrets.SONAR_ORGANIZATION }}
           -Dsonar.projectKey=${{ secrets.SONAR_PROJECT_KEY }}
           -Dsonar.sources=src/
           -Dsonar.junit.reportsPath=target/surefire-reports/ 
           -Dsonar.jacoco.reportsPath=target/jacoco.exec 
           -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
           -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/
```

To commit the changes and start the build

GitHub Actions > Run Workflow


![image](https://github.com/user-attachments/assets/734bab1a-d1e9-48b5-a231-a09fa9e3768a)


The build has been completed


![image](https://github.com/user-attachments/assets/3565f25e-a46b-4440-a378-2ebb510488d8)


## To add the QualityGate Stage (But its paid)

#### Leave this custom gate check and we can go with default QG


To create a QualityGate check in Organization and add this in project level

Sonarcloud > your organization > QualityGates > Create > Name > githubactionsQG > Save

Add Condition > On Overall Code > Quality Gate fails when "Bugs" is greater than "30" value > Add it

Then you can associate this one to your project

You can refer this link to update the workflow code


https://github.com/marketplace/actions/sonarqube-quality-gate-check


GitHub Actions > Update the main.yml

```
name: Hprofile Actions
on: workflow_dispatch
jobs:
  Testing:
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Maven test
        run: mvn test

      - name: Maven Checkstyle
        run: mvn checkstyle:checkstyle

      # Setup java 11 to be default (sonar-scanner requirement as of 5.x)
      - name: Set Java 11
        uses: actions/setup-java@v3
        with:
         distribution: 'temurin' # See 'Supported distributions' for available options
         java-version: '11'

    # Setup sonar-scanner
      - name: To Setup SonarQube
        uses: warchant/setup-sonar-scanner@v7
   
    # Run sonar-scanner
      - name: To Run SonarQube Scan
        run: sonar-scanner
           -Dsonar.host.url=${{ secrets.SONAR_URL }}
           -Dsonar.login=${{ secrets.SONAR_TOKEN }}
           -Dsonar.organization=${{ secrets.SONAR_ORGANIZATION }}
           -Dsonar.projectKey=${{ secrets.SONAR_PROJECT_KEY }}
           -Dsonar.sources=src/
           -Dsonar.junit.reportsPath=target/surefire-reports/ 
           -Dsonar.jacoco.reportsPath=target/jacoco.exec 
           -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
           -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/

      # Check the Quality Gate status.
      - name: To SonarQube Quality Gate check
        id: sonarqube-quality-gate-check
        uses: sonarsource/sonarqube-quality-gate-action@master
        # Force to fail step after specific time.
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_URL }} #OPTIONAL
```

To start the workflow > The build has been succeeded after Quality Gate status check


![image](https://github.com/user-attachments/assets/c6e43c6d-32ed-4b3e-bc9c-fb337804f9fe)


## To Create an IAM User

To create an IAM User with below policies and create accesskey/secretkey then store it somewhere safely


![image](https://github.com/user-attachments/assets/5b117f3f-1f00-4f5e-ac20-50016b26f152)


## To Create a Amazon ECR - Elastic Container Registry

To create a private repository in ECR

```
Repository name > actapp
Image tag mutability > Immutable
Create
```

![image](https://github.com/user-attachments/assets/a822118d-69ab-4638-8589-f481fc19a36e)


## To Create an Amazon RDS Instance

To create an Amazon RDS with MySQL - 8.0.33, free tier and create a accounts database

```
RDS > MySQL
Version > 8.0.33
Tier > Free
Database > accounts
Create a database
```

![image](https://github.com/user-attachments/assets/0194742c-90e1-42ff-ab3e-bd951c5e05f7)


## To launch MySQL Client Instance

To launch an Ubuntu-22 ec2 instance for MySQL Client


![image](https://github.com/user-attachments/assets/24fe08f7-e8a5-4e90-9456-4440a4bf3dad)


```
sudo -i
apt-get update -y
apt-get install mysql-client -y
mysql -h <end-point> -u <uname> -p <database>
mysql -h githubactions.cteacc4ws2sw.ap-south-1.rds.amazonaws.com -u admin -p accounts
#exit
```

To clone below link and download the db.sql file in ec2 instance using below link


https://github.com/hkhcoder/vprofile-project.git


```
git clone https://github.com/hkhcoder/vprofile-project.git
cd /root/vprofile-project/src/main/resources/
//db file shoud be available
mysql -h <end-point> -u <uname> -p <database> < /Path-to-location/db_backup.sql
mysql -h githubactions.cteacc4ws2sw.ap-south-1.rds.amazonaws.com -u admin -p accounts < /root/vprofile-project/src/main/resources/db_backup.sql
mysql -h githubactions.cteacc4ws2sw.ap-south-1.rds.amazonaws.com -u admin -p accounts
#show tables
```

![image](https://github.com/user-attachments/assets/1a56a14a-ad8d-4425-bfec-44d0475c80fc)


//Now you can terminate the MySQL Client ec2 instance


## To store the credentials in Github secrets

Your project > Settings > Secrets and variables > Actions > New repository secret

```
AWS_ACCESS_KEY_ID - **************
AWS_SECRET_ACCESS_KEY - *************
AWS_ACCOUNT_ID - 1111111111111
REGISTRY - 1111111111111.dkr.ecr.ap-south-1.amazonaws.com  //This is ecr registry url - you can get this from ecr push commands
RDS_USER - admin
RDS_PASS - *****
RDS_ENDPOINT - githubactions.cteacc4ws2sw.ap-south-1.rds.amazonaws.com
AWS_REGION - ap-south-1
```


![image](https://github.com/user-attachments/assets/bc38e84a-dc4f-4acf-bf43-e52f5faf1afc)



## To Build the Docker Image and Push the Docker Image to the Amazon ECR

Github Actions > main.yml > Edit and update the code

```
name: Hprofile Actions
on: workflow_dispatch
env:
  AWS_REGION: ap-south-1
jobs:
  Testing:
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Maven test
        run: mvn test

      - name: Checkstyle
        run: mvn checkstyle:checkstyle

      # Setup java 11 to be default (sonar-scanner requirement as of 5.x)
      - name: Set Java 11
        uses: actions/setup-java@v3
        with:
         distribution: 'temurin' # See 'Supported distributions' for available options
         java-version: '11'

    # Setup sonar-scanner
      - name: Setup SonarQube
        uses: warchant/setup-sonar-scanner@v7
   
    # Run sonar-scanner
      - name: SonarQube Scan
        run: sonar-scanner
           -Dsonar.host.url=${{ secrets.SONAR_URL }}
           -Dsonar.login=${{ secrets.SONAR_TOKEN }}
           -Dsonar.organization=${{ secrets.SONAR_ORGANIZATION }}
           -Dsonar.projectKey=${{ secrets.SONAR_PROJECT_KEY }}
           -Dsonar.sources=src/
           -Dsonar.junit.reportsPath=target/surefire-reports/ 
           -Dsonar.jacoco.reportsPath=target/jacoco.exec 
           -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
           -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/

      # Check the Quality Gate status.
      - name: SonarQube Quality Gate check
        id: sonarqube-quality-gate-check
        uses: sonarsource/sonarqube-quality-gate-action@master
        # Force to fail step after specific time.
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_URL }} #OPTIONAL   

  BUILD_AND_PUBLISH:
    needs: Testing
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Update application.properties file
        run: |
          sed -i "s/^jdbc.username.*$/jdbc.username\=${{ secrets.RDS_USER }}/" src/main/resources/application.properties
          sed -i "s/^jdbc.password.*$/jdbc.password\=${{ secrets.RDS_PASS }}/" src/main/resources/application.properties
          sed -i "s/db01/${{ secrets.RDS_ENDPOINT }}/" src/main/resources/application.properties

      - name: Build & Upload image to ECR
        uses: appleboy/docker-ecr-action@master
        with:
         access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
         secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
         registry: ${{ secrets.REGISTRY }}
         repo: actapp
         region: ${{ env.AWS_REGION }}
         tags: latest,${{ github.run_number }}
         daemon_off: false
         dockerfile: ./Dockerfile
         context: ./

```

To start the build and check the result - Now two jobs are running.


![image](https://github.com/user-attachments/assets/b63f7a74-4cd2-4354-9204-65995f43d41b)


The build has been succeeded


![image](https://github.com/user-attachments/assets/2c4317ce-eb86-46e9-b265-6f29bc1d17d4)


Docker image has been pushed in to the ECR repository


![image](https://github.com/user-attachments/assets/459f0397-cad4-4007-9e1b-9c1a803c44ce)



## To create an Amazon ECS Cluster

AWS > ECS Cluster > Create

```
Cluster name - vpro-act
Default namespace - vpro-act
AWS Infrastructure - Fargate
Tag > Name > vpro-act
Create a Cluster
```


![image](https://github.com/user-attachments/assets/434f1c99-c3fa-481d-a85c-f206ea621015)


#### To create a new Task Definition

AWS > ECS Cluster > Task definition > new Task definition

```
Task definition family > vproapp-act-tdef
Launch type > AWS fargate
Operating system/Architecture > Linux/X86_64
CPU > 1 and Memory > 2 GB
Task role > -
Task execution role - Create new role
Container name > vproapp
Image URI > 590183829524.dkr.ecr.ap-south-1.amazonaws.com/actapp
Essential container - yes
Container port > 8080
Protocol > TCP
Port name > vproapp-8080-tcp
App protocol > HTTP
Log collection > Use log collection > Amazon Cloudwatch
Create
```

![image](https://github.com/user-attachments/assets/f5c9a012-ae66-4f0d-b02f-4468ce462f75)


#### To Create a new Service in ECS

AWS > ECS > Your Cluster > Create a new service

```
Compute options > Capacity provider strategy
Capacity provider > Fargate
Platform version > LATEST
Deployment configuration > Application type > Service
Family > vproapp-act-tdef > Revision - 1
Service name > vproapp-act-svc
Service type > replica
Desired tasks > 1
Deployment type > Rolling update
Deployment failure detection > unselect > Use the Amazon ECS deployment circuit breaker
Networking > Choose your VPC and all subnets
Security group > Exisiting or new
PublicIP - Turned on
Loadbalancing > Use Loadbalancing > ALB
Conatainer > vproapp 8080:8080
Application loadbalancer > Create a new loadbalancer
Loadbalancer name > vproapp-act-elb
Create a new listener > Port > 80 > Protocol > HTTP
Target group > Create new Target group > vproapp-act-tg
Protocol > HTTP
Deregistration delay > 120
Healthchecks protocol > HTTP
Healthcheck path > /login
Tag > Name > vproapp-act-svc
Create a Service
```

Service has been created


![image](https://github.com/user-attachments/assets/57aff455-c0b0-4ce7-b92b-0eed6e5d9a66)


If you check with the ALB URL, then you can access the application


![image](https://github.com/user-attachments/assets/e8a5ec8d-fd06-4358-8d95-7e34a099204e)


#### To place the ECS Taskdefinition json in Github

You can copy the json from AWS > ECS > Task definition > Copy the json

Place it > Github > project > hprofile/aws-files/taskdeffile.json


![image](https://github.com/user-attachments/assets/075f6359-581c-4963-bfe9-2b73fa6d2dcc)


#### To update the main yaml file

GitHub Actions > project > main.yml

```
name: Hprofile Actions
on: workflow_dispatch
env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: actapp
  ECS_SERVICE: vproapp-act-svc
  ECS_CLUSTER: vpro-act
  ECS_TASK_DEFINITION: aws-files/taskdeffile.json
  CONTAINER_NAME: vproapp
jobs:
  Testing:
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Maven test
        run: mvn test

      - name: Maven Checkstyle
        run: mvn checkstyle:checkstyle

      # Setup java 11 to be default (sonar-scanner requirement as of 5.x)
      - name: To Setup Java 11
        uses: actions/setup-java@v3
        with:
         distribution: 'temurin' # See 'Supported distributions' for available options
         java-version: '11'

    # Setup sonar-scanner
      - name: To Setup SonarQube
        uses: warchant/setup-sonar-scanner@v7
   
    # Run sonar-scanner
      - name: Run SonarQube Scan
        run: sonar-scanner
           -Dsonar.host.url=${{ secrets.SONAR_URL }}
           -Dsonar.login=${{ secrets.SONAR_TOKEN }}
           -Dsonar.organization=${{ secrets.SONAR_ORGANIZATION }}
           -Dsonar.projectKey=${{ secrets.SONAR_PROJECT_KEY }}
           -Dsonar.sources=src/
           -Dsonar.junit.reportsPath=target/surefire-reports/ 
           -Dsonar.jacoco.reportsPath=target/jacoco.exec 
           -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
           -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/

      # Check the Quality Gate status.
      - name: SonarQube Quality Gate check
        id: sonarqube-quality-gate-check
        uses: sonarsource/sonarqube-quality-gate-action@master
        # Force to fail step after specific time.
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_URL }} #OPTIONAL   

  BUILD_AND_PUBLISH:
    needs: Testing
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Update application.properties file
        run: |
          sed -i "s/^jdbc.username.*$/jdbc.username\=${{ secrets.RDS_USER }}/" src/main/resources/application.properties
          sed -i "s/^jdbc.password.*$/jdbc.password\=${{ secrets.RDS_PASS }}/" src/main/resources/application.properties
          sed -i "s/db01/${{ secrets.RDS_ENDPOINT }}/" src/main/resources/application.properties

      - name: Build & Upload image to ECR
        uses: appleboy/docker-ecr-action@master
        with:
         access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
         secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
         registry: ${{ secrets.REGISTRY }}
         repo: actapp
         region: ${{ env.AWS_REGION }}
         tags: latest,${{ github.run_number }}
         daemon_off: false
         dockerfile: ./Dockerfile
         context: ./
  
  
  Deploy:
    needs: BUILD_AND_PUBLISH
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Fill in the new image ID in the Amazon ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: ${{ env.ECS_TASK_DEFINITION }}
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ secrets.REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ github.run_number }}

      - name: Deploy Amazon ECS task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
 ```

To start the build and check all the stages


![image](https://github.com/user-attachments/assets/ab9c5c31-15d3-46f4-abd4-48f8b2cb41f2)


My build has been succeeded. You can access the ALB URL
        
  
![image](https://github.com/user-attachments/assets/e16c7e07-1fae-4f0e-8845-c5a580b64a23)


The image has been updated in ECR

![image](https://github.com/user-attachments/assets/670978f8-56c1-4825-84df-70dc50276ed3)


The same image version has been deployed in AWS ECS task definition


![image](https://github.com/user-attachments/assets/30912dac-f4f0-4349-bbc5-33d17d52f2f5)








