![image](https://github.com/user-attachments/assets/e1357608-3d2b-4cea-85e7-6e81d3513670)# GitOps to deploy application in Amazon Elastic Kubernetes Cluster using GitHub Actions Workflows, Terraform, Maven, SonarCloud, Docker, ECR, Helm Charts and EKS



![image](https://github.com/user-attachments/assets/1a96e515-4047-4a26-bbc6-7085ba9c117e)



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


