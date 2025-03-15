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







