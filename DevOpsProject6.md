# Kubernetes Project

## Deploy a Kubernetes Cluster using KOPS

#### Launch an EC2 instance

To launch Ubuntu-24 EC2 instance with T2.Micro for KOPS operation


![image](https://github.com/user-attachments/assets/ae993064-d59a-4d7d-be1a-a6f68c08cc0a)


To assign IAM Role with Administrator access for KOPS instance. Because this KOPS will communicate with lot of AWS services such as Autoscaling, Route53, S3 bucket...


![image](https://github.com/user-attachments/assets/3aa6550b-42e2-4455-a915-97f7a230494d)


#### Install AWS CLi

SSH to KOPS instance

```
sudo -i
ssh-keygen
ls -lh ~/.ssh/
//To note the public key file name (id_ed25519.pub)
snap install aws-cli --classic
aws --version
```

#### Install KOPS

You can refer - https://kops.sigs.k8s.io/getting_started/install/

```
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops
sudo mv kops /usr/local/bin/kops
kops version
```

#### Install kubectl

You can refer - https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

#### Create a S3 Bucket

All the cluster information should stored in this S3 bucket. Whenever we use the kops command such as create, validate and delete then we should use s3 bucket


![image](https://github.com/user-attachments/assets/0fca86a1-0e83-4f4c-ab22-c0596928ed21)


#### To create a subdomain in Route53

To create a subdomain in Route53 and map the NS in root domain registrar

![image](https://github.com/user-attachments/assets/9c698d1a-6334-4cb9-9e03-898d13b54773)


#### To create and update a kops cluster

```
kops create cluster --name=kubepro.demo.com --state=s3://kopsstate744 --zones=ap-south-1a,ap-south-1b --node-count=2 --node-size=t3.small --control-plane-size=t3.medium --dns-zone=kubepro.demo.com --node-volume-size=12 --control-plane-volume-size=12 --ssh-public-key ~/.ssh/id_ed25519.pub
kops update cluster --name=kubepro.demo.com --state=s3://kopsstate744 --yes --admin
kops validate cluster --name=kubepro.demo.com --state=s3://kopsstate744
ls -la .kube
cat .kube/config
```

My cluster has been created


![image](https://github.com/user-attachments/assets/bb596b5d-ad57-44dc-9069-bce418c56eaa)


![image](https://github.com/user-attachments/assets/629f87fe-a4ec-4541-9aa6-16d7d1bc9d7a)


#### To delete the kops cluster

```
kops delete cluster --name=kubepro.demo.com --state=s3://kopsstate744 --yes
```


# To Deploy a Amazon EKS Cluster using EKSCTL


#### Launch an EC2 instance

To launch Ubuntu-24 EC2 instance with T2.Micro for KOPS operation


![image](https://github.com/user-attachments/assets/ed93e1b7-3be8-4258-b3e2-bed74b3d3e75)


To assign IAM Role with Administrator access for EKSCTL instance

#### Install AWS CLi

SSH to EKSCTL instance

```
sudo -i
snap install aws-cli --classic
aws --version
```

#### Install kubectl

You can refer - https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```


#### Install EKSCTL

You can refer - https://eksctl.io/installation/

```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

![image](https://github.com/user-attachments/assets/b65229ce-7e7a-491f-b754-ddaaadfa7a0b)


#### Install EKS Cluster


You can use below link to download the script to launch cluster

```
https://github.com/kohlidevops/DevOpsProjects/blob/main/eks-cluster-setup.sh
sudo -i
sh eks-cluster-setup.sh
```

EKS Cluster has been created with their own VPC and master node managed by Amazon


![image](https://github.com/user-attachments/assets/349c63a2-94d9-4a96-88bf-e2022aaf2062)


You can see the worker nodes which is managed by you


![image](https://github.com/user-attachments/assets/38014ecf-bf4a-427a-9329-d6ee0a966c09)


If you execute kubectl command from EKSCTL instance

```
kubectl get nodes
kubectl get pods -o wide
```

![image](https://github.com/user-attachments/assets/fbd38931-5f8b-4b04-bc7f-92c6ccfdb1ca)


#### To delete the EKSCTL Cluster

To delete the EKS cluster entirely using below command

```
sudo -i
eksctl delete cluster vprofile-eks-cluster --region ap-south-1
```


![image](https://github.com/user-attachments/assets/71bda923-403d-4d58-8114-880bd13fa646)


The cluster has been removed from AWS EKS
