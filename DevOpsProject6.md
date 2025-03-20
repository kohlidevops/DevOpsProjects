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


## Architecture Diagram


![image](https://github.com/user-attachments/assets/7753fa52-193a-4f15-a476-17e4a12e396c)


**1. Application Load Balancer (ALB)**

The ALB distributes incoming traffic from users to the backend services in a balanced way, ensuring no one service is overloaded.It acts as the entry point for all the incoming traffic.


**2. Ingress**

Ingress manages external access to services within the Kubernetes cluster, typically HTTP or HTTPS traffic. It provides routing rules to connect the outside world to the services. Acts as a gateway within Kubernetes, forwarding the incoming requests to the appropriate service (like TomcatService). 


**3. TomcatService**

This service is responsible for routing requests to the Tomcat pods in the Kubernetes cluster. TomcatService allows clients to access the Tomcat application.


**4. Tomcat Pod**

Represents a pod running an instance of the Tomcat web server inside the Kubernetes cluster. Tomcat is used to host and run Java-based applications and APIs.


**5. RabbitMQ Pod**

RabbitMQ is a message broker that helps in asynchronous communication between services.


**6. Memcache Pod**

Memcached is an in-memory key-value store used for caching data.


**7. DB Pod**

Represents the database pod where the actual MySQL database resides. The DB Pod stores the persistent data of the application. The data is saved in /var/lib/mysql, which is mounted to an external storage (Amazon EBS) for persistence.


**8. PersistentVolumeClaim (PVC)**

PVC is a request for storage by a user, which is fulfilled by a PersistentVolume (PV). It allows pods to request storage resources. The PVC is used to persist the MySQL data in the database pod, ensuring that the data is not lost when the pod is destroyed or rescheduled.


**9. StorageClass**

StorageClass defines the type of storage being provisioned for PersistentVolumes. It is associated with Amazon EBS and provides the backend storage to satisfy the PVC requests. It allows dynamic provisioning of storage volumes as needed by the cluster.


**10. Amazon EBS (Elastic Block Store)**

Amazon EBS is used to provide block storage volumes for use with EC2 instances or Kubernetes pods.


**11. Secret**

Secrets are used in Kubernetes to store sensitive information such as passwords, tokens, and keys. The Secret is likely being used to store sensitive credentials, such as database passwords, access tokens, or certificates, which are accessed by the services and pods securely.


## Source code repo

To fork the below repo with unselect -> main branch only


```
https://github.com/kohlidevops/vprofile-project-kube/tree/kubeapp

select > kubeapp branch
```

In kubedefs folder, we have all the manifest yaml file for k8 application

You can refer the yaml files

secret.yaml > dbvpc.yaml > dbdeploy.yaml > dbservice.yaml > mcdep.yaml > mcservice.yaml > rmqdeploy.yaml > rmqservice.yaml > appdeploy.yaml > appservice.yaml > appingress.yaml


![image](https://github.com/user-attachments/assets/ae55617d-dbeb-45bd-9e29-1915754bae76)



## Create a Kubernetes Cluster using KOPS

SSH to Kops instance

```
sudo -i
Install kubectl, awscli, kops in kops instance
Create a HostedZone
Create a S3 bucket
Create a ssh-keygen in kops instance as root user

kops create cluster --name=kubevpro.demo.com --state=s3://kopsstate744 --zones=ap-south-1a,ap-south-1b --node-count=2 --node-size=t3.small --control-plane-size=t3.medium --dns-zone=kubevpro.demo.com --node-volume-size=12 --control-plane-volume-size=12 --ssh-public-key ~/.ssh/id_ed25519.pub
kops update cluster --name=kubevpro.demo.com --state=s3://kopsstate744 --yes --admin
kops validate cluster --name=kubevpro.demo.com --state=s3://kopsstate744
ls -la .kube
cat .kube/config
```

The master and worker node has been created 


![image](https://github.com/user-attachments/assets/dffacd54-4d2f-47d0-9b74-60bcc110b755)


Austoscaling has been created for every node

All the cluster information has been stored in S3 bucket


![image](https://github.com/user-attachments/assets/2ba17da4-cf03-4265-90be-20565f980d25)


The records are created in AWS Route53 Hosted Zone


![image](https://github.com/user-attachments/assets/2ecc9c0b-ce8d-4bef-8dff-513150d579fe)


If I validate my cluster, then my cluster are up and running


![image](https://github.com/user-attachments/assets/232182b2-0cb2-4e12-8807-26f73b813f28)


We can check with nodes status and check all the default pods are running or not


```
kubectl get nodes
kubectl get pods -A
```


![image](https://github.com/user-attachments/assets/b7cef7ae-6e01-4368-b54d-239cf33baf2b)


### To setup the repo for deployment

To clone the repo with specific branch in local machine

```
git clone --branch kubeapp --single-branch https://github.com/kohlidevops/vprofile-project-kube.git
cd vprofile-project-kube
ls -lh
```


![image](https://github.com/user-attachments/assets/f1aa7522-1b43-48f7-805a-c08ae68c1bc5)


Keep the necessary file as we are not doing build, test like. - So keep the below folders and files

```
Docker-files
docker-compose.yml
kubedefs
```


To create a new repo named kubevpro in Github repo and upload the above files


![image](https://github.com/user-attachments/assets/38b6377c-ffe5-4a2d-84b1-0213242a2b42)


To clone the new repo to the kops instance

```
git clone https://github.com/kohlidevops/kubevpro.git
ls -lh
```


![image](https://github.com/user-attachments/assets/70c8a1ba-ae74-48c2-b9b8-4b20c39d56cc)


## To create a NGINX Ingress Controller

To create a NGINX ingress controller for AWS which is manage the Loadbalancer - For this to deploy in kops instance

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
kubectl get ns
kubectl get pods -n ingress-nginx
```


![image](https://github.com/user-attachments/assets/25a13443-923a-4d1f-aeac-760c952e28e0)


It will provisioning the Network Loadbalancer for us


![image](https://github.com/user-attachments/assets/fb3d728d-df99-4a7c-89eb-1ab13552b0d2)


## Deploy the PVC

SSH to kops instance

```
cd kubevpro/kubedefs/
ls -lh
kubectl apply -f dbpvc.yaml
kubectl get pvc
```

![image](https://github.com/user-attachments/assets/c4b8589a-32f7-4949-b392-62db34ac184f)


If you check the AWS EBS volume, it should be availabe state


![image](https://github.com/user-attachments/assets/abd65abd-7d1b-43c2-b7a1-b03558038f31)


##  Deploy all the remaining manifest files

```
cd kubevpro/kubedefs/
kubectl apply -f .
```


![image](https://github.com/user-attachments/assets/25d2f14a-95ae-4b7c-8d20-b03c0c187b96)


To check pods, deploy and svc

```
kubectl get pods
kubectl get deploy
kubec6tl get svc
```


![image](https://github.com/user-attachments/assets/ca1dd45b-6697-49c0-9ba6-de3a633132f7)


If you want describe the pod

```
kubectl describe pod <pod-name>
```


All the pods are running as we expected and EBS dynamic provisioning volumes are now in-use means its attached to the node


![image](https://github.com/user-attachments/assets/925097c0-9647-49bc-aeae-517f3ccf4bf2)


To describe the service


```
kubectl describe svc vproapp-service
```


![image](https://github.com/user-attachments/assets/a15bb6fe-21ec-4592-9d94-55a3ce0e94ad)



To check the ingress

```
kubect get ingress
```


![image](https://github.com/user-attachments/assets/a7ca2e1c-dc5f-4cae-8c7e-5d0b4d6a4018)


### To map the service endpoint in Route53


```
kubectl describe ingress <ingress-name>
//map the domain with the alb endpoint
```


If you check with domain name, it should be accessible over the browser


![image](https://github.com/user-attachments/assets/f623057d-7a45-4e38-8129-109ea76e1bf9)


Even we can able to login

```
uname - admin_vp
pwd - admin_vp
```


![image](https://github.com/user-attachments/assets/2ee9668d-69e3-48c6-a09d-9bdba2748149)



