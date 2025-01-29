# To Setup DevOps Project Using Terraform, Ansible, Jenkins, Maven, SonarQube, Docker, Helm Charts, Kubernetes, Promotheus and Grafana

## Pre-requisites

1. AWS CloudShell

2. Install and configure AWS CLI on Cloudshell

Refer - https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

3. Install Terraform on Cloudshell

Refer - https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli

```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform
terraform -help
```

4. Github repository

source code repo - https://github.com/kohlidevops/tweet-trend-new

lab code repo - https://github.com/kohlidevops/devops-workshop

## Terraform to provisioning resources

**1. To create a simple Ec2 instance using terraform**

https://github.com/kohlidevops/devops-workshop/blob/main/terraform_code/V1-EC2.tf

To change the values as per your environment and run the below command

```
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy  #once your checked with your aws console, then you can destroy it
```

<img width="793" alt="image" src="https://github.com/user-attachments/assets/da54de5a-82ec-4ad9-bff5-4a99af570b03" />

**2. To create a Ec2 instance along with Security group**

https://github.com/kohlidevops/devops-workshop/blob/main/terraform_code/V2-EC2.tf

```
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy  #once your checked with your aws console, then you can destroy it
```

**3. To create a VPC and Ec2 instance along with Security group**

https://github.com/kohlidevops/devops-workshop/blob/main/terraform_code/V3-EC2-With_VPC.tf

```
terraform init
terraform validate
terraform plan
terraform apply
```

Dont destroy this template

**4. To create a 3 more Ec2 instances using for each loop**

https://github.com/kohlidevops/devops-workshop/blob/main/terraform_code/V4-EC2-With_VPC_for_each.tf

To change the AMI as ubuntu-22

```
terraform init
terraform validate
terraform plan
terraform apply
```

Now exisiting instance will be deleted then create a new 3 Ec2 instances

<img width="797" alt="image" src="https://github.com/user-attachments/assets/6bdd8b3e-631e-4645-95cd-0a9d4f212352" />






