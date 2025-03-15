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


