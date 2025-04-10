# GitOps to deploy application in Amazon Elastic Kubernetes Cluster using GitHub Actions Workflows, Terraform, Maven, SonarCloud, Docker, ECR, Helm Charts and EKS



![image](https://github.com/user-attachments/assets/cb5778dc-2b86-47bf-810e-283cf9586640)




#### To fork the below repo to your GitHub

To fork the below 2 repos with uncheck the branch named "main"


```
https://github.com/kohlidevops/iac-vprofile.git
https://github.com/kohlidevops/vprofile-action.git
```


#### To create a SSH Keygen

To generate the ssh-keygen in you local system > open bash commandline

```
cd ~
ssh-keygen.exe
Enter the file name to save the key:  /C/Users/lenovoS340/.ssh/actions
ls -lh .ssh/
//actions and actions.pub key available
```

To copy/paste this pub key to GitHub

GitHub > Your profile > Settings > SSH and GPG keys > Add SSH > Key type > Authentication key > paste it > save the content


To export the SSH command

export GIT_SSH_COMMAND="ssh -i ~/.ssh/actions"


To create a directory and clone the repo

```
mkdir D:\Devops-Project\devops4sure
cd D:\Devops-Project\devops4sure
git clone git@github.com:kohlidevops/iac-vprofile.git
git clone git@github.com:kohlidevops/vprofile-action.git
ls -lh
cd iac-vprofile
git config core.sshCommand "ssh -i ~/.ssh/actions -F /dev/null"
cd vprofile-action
git config core.sshCommand "ssh -i ~/.ssh/actions -F /dev/null"
cd iac-profile
git config --global user.name kohlidevops
git config --global user.email latchudevops1@gmail.com
cd vprofile-action
git config --global user.name kohlidevops
git config --global user.email latchudevops1@gmail.com
cd ..
cp -r iac-profile main-iac
cd iac-profile
```


![image](https://github.com/user-attachments/assets/6285612a-6b3c-40e6-8613-4aaaeec466cf)


![image](https://github.com/user-attachments/assets/ee3a8d99-0c92-4e32-b50e-0c7dd57bbf5a)


To make ensure the brach is stage in iac-profile repo

```
cd iac-vprofile
git branch -a
git checkout stage
//We always work in stage branch
//When we fine with this branch, then we merge this branch with main branch
```


![image](https://github.com/user-attachments/assets/4ab3e2ef-3173-4d51-b595-158fd2c7ea0d)


#### To Setup Secrets in GitHub

To create an IAM user with Administrator access and create accesskey for this


![image](https://github.com/user-attachments/assets/c48ad362-f42c-4e83-b5cc-5eb98ecfc3f0)


To store this access key in iac-vprofile and vprofile-action of GitHub repo


![image](https://github.com/user-attachments/assets/402ae975-b5a4-4859-84d7-7aa6804f60ee)


To create a S3 bucket named "latchuprofileactions" for terraform tfstate file


![image](https://github.com/user-attachments/assets/3294ab2d-e409-4551-b4bf-7c986add65c2)


To store the bucket name in iac-vprofile secrets of GitHub repo


![image](https://github.com/user-attachments/assets/5029eed2-ad63-4a76-876a-ec3bbac4f592)


To create a private repo named vprofileapp in AWS ECR

ECR > Private repo > name > vprofileapp > create


![image](https://github.com/user-attachments/assets/672ffeda-1abf-4a49-ae8a-03120b6f4e18)


To copy the registry URL in secrets of the vprofile-action repo


![image](https://github.com/user-attachments/assets/ad772886-1174-4571-8aa9-22a5e833790f)


Registry URL should be in secrets - 590183829524.dkr.ecr.us-east-1.amazonaws.com



#### To setup Terraform code for staging


Navigate to below repo and switch to stage branch


https://github.com/kohlidevops/iac-vprofile/tree/stage/terraform


Then do the necessary changes like region, cluster name, and bucket name in variables.tf and terraform.tf file


### Staging Workflow for Terraform code


Navigate to your repo and switch to stage branch


https://github.com/kohlidevops/iac-vprofile/tree/stage


To create a workflow for staging


Create a folder > .github/workflows/terraform.yml


![image](https://github.com/user-attachments/assets/b558ef7b-5968-4a05-8159-3c9e4711cacc)


To update the terraform.yml


```
name: "Vprofile with IAC"

on:
  push:
    branches:
      - main
      - stage
    paths:
      - terraform/**
  pull_request:
    branches:
      - main
    paths:
      - terraform/**

env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  BUCKET_TF_STATE: ${{ secrets.BUCKET_TF_STATE }}
  AWS_REGION: us-east-1
  EKS_CLUSTER: vprofile-eks

jobs:
  terraform:
    name: "Apply Terraform Code Changes"
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        working-directory: ./terraform
    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Setup Terraform with specified version on the runner
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: "1.6.3"

      - name: Terraform init
        id: init
        run: terraform init -backend-config="bucket=$BUCKET_TF_STATE"

      - name: Terraform format
        id: fmt
        run: terraform fmt -check

      - name: Terraform validate
        id: validate
        run: terraform validate

      - name: Terraform plan
        id: plan
        run: terraform plan -no-color -input=false -out planfile
        continue-on-error: true

      - name: Terraform plan status
        if: steps.plan.outcome == 'failure'
        run: exit 1
```

Do some changes in terraform/terraform.tf files (branch should be "stage")


My Stage job has been succeeded


![image](https://github.com/user-attachments/assets/1526b251-e8c0-49bd-96cc-24f65f3cc5c9)



### Main Workflow for Terraform code

Update the terraform.yml to add terraform apply, configure aws credential, get kubeconfig file and install ingress controller in stage branch


```
name: "Vprofile with IAC"

on:
  push:
    branches:
      - main
      - stage
    paths:
      - terraform/**
  pull_request:
    branches:
      - main
    paths:
      - terraform/**

env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  BUCKET_TF_STATE: ${{ secrets.BUCKET_TF_STATE }}
  AWS_REGION: us-east-1
  EKS_CLUSTER: vprofile-eks

jobs:
  terraform:
    name: "Apply Terraform Code Changes"
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        working-directory: ./terraform
    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Setup Terraform with specified version on the runner
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: "1.6.3"

      - name: Terraform init
        id: init
        run: terraform init -backend-config="bucket=$BUCKET_TF_STATE"

      - name: Terraform format
        id: fmt
        run: terraform fmt -check

      - name: Terraform validate
        id: validate
        run: terraform validate

      - name: Terraform plan
        id: plan
        run: terraform plan -no-color -input=false -out planfile
        continue-on-error: true

      - name: Terraform plan status
        if: steps.plan.outcome == 'failure'
        run: exit 1

      - name: Terraform Apply
        id: apple
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply -auto-approve -input=false -parallelism=1 planfile

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Get Kube config file
        id: getconfig
        if: steps.apple.outcome == 'success'
        run: aws eks update-kubeconfig --region ${{ env.AWS_REGION }} --name ${{ env.EKS_CLUSTER }}

      - name: Install Ingress controller
        if: steps.apple.outcome == 'success' && steps.getconfig.outcome == 'success'
        run: kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```

Commit the changes - It wont trigger, becuase we are changing in terraform.yml file, not in terraform folder

Do some changes in terraform.tf code and check - workflow will trigger and it will run upto terraform plan, but it wont run terraform apply. Because we are updating the code in stage branch - so it will skip the terraform apply.


![image](https://github.com/user-attachments/assets/ea14d245-1633-4105-89e4-109ab2e4e5c5)



#### How to lock the main branch in iac-vprofile (But ignore this step as of now)


Github repo > iac-vprofile > main branch > settings > branches > Add classic branch protection rule > Branch name pattern > main > Choose > Require a pull request before merging  and Require approvals > create


#### How to merge the code from stage to main

Go to - /c/Users/lenovoS340/D:Devops-Projectdevops4sure/main-iac and make ensure you are in main branch

```
git pull
ls
git checkout stage
git checkout main
git merge stage
git status
git push origin main
```

To check on Github repo > iac-vprofile > Actions > Workflow triggered and this is main branch, so terraform apply will start and resources are deploying now.


My workflow has been completed


![image](https://github.com/user-attachments/assets/c8e367bc-0b0f-4ee8-92f2-54b4ef4143c9)


EKS cluster has been created with node groups


![image](https://github.com/user-attachments/assets/1dc66ea3-9733-4552-ba9a-99cee33778d2)


Node group instances are available with their own autoscaling


![image](https://github.com/user-attachments/assets/6df67718-b7c6-4729-acce-a789c5583e89)


Ingress controller has been created means Loadbalancer was launched


![image](https://github.com/user-attachments/assets/6018b334-00a1-4fa1-820e-711f7a6d1385)



## Workflow for application code repository


### SonarCloud setup

To login sonarcloud with Github login


![image](https://github.com/user-attachments/assets/b2c1a086-7cf2-4f73-b10c-00f1327c3944)



To create a new organization > create one manually

```
Name > vprofile-actions8
Key > vprofile-actions8
Choose > free plan
Create Organization
Analyze a new project
Display name > vproapp8
Project key > vproapp8
Project visibility > Public
Next
The new code for this project will be based on > Previous version
Create Project
```


To create a token

My account > Security > Generate token > name > gitops > Generate it

```
e2cf4ba73b300ae35b391fc5f21d2f4725ce741f
```

Github > vprofile-action (application code repo) > main branch > settings > secrets and variables > actions > new repository secrets > 


```
SONAR_TOKEN - e2cf4ba73b300ae35b391fc5f21d2f4725ce741f
SONAR_ORGANIZATION - vprofile-actions8
SONAR_PROJECT_KEY - vproapp8
SONAR_URL - https://sonarcloud.io
save it
```


![image](https://github.com/user-attachments/assets/9437212e-0e03-4376-b644-7b79c0a55c2e)



To create a main.yml in vprofile-action (main branch) .github/workflows/main.yml

```
name: vprofile actions
on: workflow_dispatch
env:
  AWS_REGION: us-east-2
  ECR_REPOSITORY: vprofileapp
  EKS_CLUSTER: vprofile-eks

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
```

To start this build as manually - Gitub Actions > Run > workflow


My workflow has been completed


![image](https://github.com/user-attachments/assets/6480c1b9-34bc-4f49-8998-fee057bf044d)



### To update the code for Docker build and push to ECR


To update the main.yml code and start the workflow


```
name: vprofile actions
on: workflow_dispatch
env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: vprofileapp
  EKS_CLUSTER: vprofile-eks

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

      
  BUILD_AND_PUBLISH:   
    needs: Testing
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Build & Upload image to ECR
        uses: appleboy/docker-ecr-action@master
        with:
         access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
         secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
         registry: ${{ secrets.REGISTRY }}
         repo: ${{ env.ECR_REPOSITORY }}
         region: ${{ env.AWS_REGION }}
         tags: latest,${{ github.run_number }}
         daemon_off: false
         dockerfile: ./Dockerfile
         context: ./
```

To run the workflow


The build and push docker images has been done



![image](https://github.com/user-attachments/assets/4910e152-6527-4251-adbf-893515fd2813)


If you check with ECR repository


![image](https://github.com/user-attachments/assets/5d1ede54-0eea-4576-995a-8bf656f3c6e6)




My job has been succeeded


### To add the Kubernetes deployment job


Github repo > vprofile-action > main > .github/workflows > main.yml

```
name: vprofile actions
on: workflow_dispatch
env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: vprofileapp
  EKS_CLUSTER: vprofile-eks

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

      
  BUILD_AND_PUBLISH:   
    needs: Testing
    runs-on: ubuntu-latest
    steps:
      - name: Code checkout
        uses: actions/checkout@v4

      - name: Build & Upload image to ECR
        uses: appleboy/docker-ecr-action@master
        with:
         access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
         secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
         registry: ${{ secrets.REGISTRY }}
         repo: ${{ env.ECR_REPOSITORY }}
         region: ${{ env.AWS_REGION }}
         tags: latest,${{ github.run_number }}
         daemon_off: false
         dockerfile: ./Dockerfile
         context: ./

  DeployToEKS:
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

      - name: Get Kube config file
        run: aws eks update-kubeconfig --region ${{ env.AWS_REGION }} --name ${{ env.EKS_CLUSTER }}

      - name: Print config file
        run: cat ~/.kube/config

      - name: Login to ECR
        run: kubectl create secret docker-registry regcred --docker-server=${{ secrets.REGISTRY }} --docker-username=AWS  --docker-password=$(aws ecr get-login-password) 

      - name: Deploy Helm
        uses: bitovi/github-actions-deploy-eks-helm@v1.2.8
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
          cluster-name: ${{ env.EKS_CLUSTER }}
          #config-files: .github/values/dev.yaml
          chart-path: helm/vprofilecharts
          namespace: default
          values: appimage=${{ secrets.REGISTRY }}/${{ env.ECR_REPOSITORY }},apptag=${{ github.run_number }}
          name: vprofile-stack
```

We are using Helm charts to deploy the k8 manifest yaml

The charts are available in repo - vprofile-action/helm/vprofilecharts

The domain should be updated in helm/vprofilecharts/templates/vproingress.yaml


To start the workflow


![image](https://github.com/user-attachments/assets/7934f109-00a9-4f8c-b862-a9f2ab677ca2)



Once done, then map the ALB URL with domain name - then you can view the site


![image](https://github.com/user-attachments/assets/d092ad08-133b-4f30-b570-d8a60bae6429)



### To Clean up

Remove the ALB and TG

Remove the EKS node group

Remove the EKS

Remove the ECR

Remove the IAM user







