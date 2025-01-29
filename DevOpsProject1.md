# To Setup DevOps Project Using Terraform, Ansible, Jenkins, Maven, SonarQube, Docker, Helm Charts, Kubernetes, Promotheus and Grafana

**Pre-requisites**

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

