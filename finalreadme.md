***********read the file 10-ECR-Elastic-Container-Registry-and-EKS/README.md ************
# AWS ECR - Elastic Container Registry Integration & EKS

## Step-01: What are we going to learn?
- We are going build a Docker image 
- Push to ECR Repository
- Update that ECR Image Repository URL in our Kubernetes Deployment manifest
- Deploy to EKS
- Kubernetes Deployment, NodePort Service, Ingress Service and External-DNS will be used to depict a full-on deployment
- We will access the ECR Demo Application using registered dns `http://ecrdemo.kubeoncloud.com`

## Step-02: ECR Terminology
 - **Registry:** An  ECR registry is provided to each AWS account; we can create image repositories in our registry and store images in them. 
- **Repository:** An ECR image repository contains our Docker images. 
- **Repository policy:** We can control access to our repositories and the images within them with repository policies. 
- **Authorization token:** Our Docker client must authenticate to Amazon ECR registries as an AWS user before it can push and pull images. The AWS CLI get-login command provides us with authentication credentials to pass to Docker. 
- **Image:** We can push and pull container images to our repositories.  

## Step-03: Pre-requisites
- Install required CLI software on your local desktop
   - **Install AWS CLI V2 version**
      - We have taken care of this step as part of [01-EKS-Create-Clusters-using-eksctl](/01-EKS-Create-Cluster-using-eksctl/01-01-Install-CLIs/README.md)
      - Documentation Reference: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html
   - **Install Docker CLI** 
      - We have taken of Docker local desktop installation as part of [Docker Fundamentals](https://github.com/stacksimplify/docker-fundamentals/tree/master/02-Docker-Installation) section 
      - Docker Desktop for MAC: https://docs.docker.com/docker-for-mac/install/
      - Docker Desktop for Windows: https://docs.docker.com/docker-for-windows/install/
      - Docker on Linux: https://docs.docker.com/install/linux/docker-ce/centos/

   - **On AWS Console**
      - We have taken care of this step as part of [01-EKS-Create-Clusters-using-eksctl](/01-EKS-Create-Cluster-using-eksctl/01-01-Install-CLIs/README.md)
      - Create Authorization Token for admin user if not created
      - **Configure AWS CLI with Authorization Token**
```
aws configure
AWS Access Key ID: ****
AWS Secret Access Key: ****
Default Region Name: us-east-1
```   

## Step-04: Create ECR Repository
- Create simple ECR repository via AWS Console 
   - Repository Name: aws-ecr-kubenginx
   - Tag Immutability: Enable
   - Scan on Push: Enable
- Explore ECR console. 
- **Create ECR Repository using AWS CLI**
```
aws ecr create-repository --repository-name aws-ecr-kubenginx --region us-east-1
aws ecr create-repository --repository-name <your-repo-name> --region <your-region>
```

## Step-05: Create Docker Image locally
- Navigate to folder **10-ECR-Elastic-Container-Registry\01-aws-ecr-kubenginx** from course github content download. 
- Create docker image locally
- Run it locally and test
```
# Build Docker Image
docker build -t <ECR-REPOSITORY-URI>:<TAG> . 
docker build -t 180789647333.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-kubenginx:1.0.0 . 

# Run Docker Image locally & Test
docker run --name <name-of-container> -p 80:80 --rm -d <ECR-REPOSITORY-URI>:<TAG>
docker run --name aws-ecr-kubenginx -p 80:80 --rm -d 180789647333.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-kubenginx:1.0.0

# Access Application locally
http://localhost

# Stop Docker Container
docker ps
docker stop aws-ecr-kubenginx
docker ps -a -q
```

## Step-06: Push Docker Image to AWS ECR
- Firstly, login to ECR Repository
- Push the docker image to ECR
- **AWS CLI Version 2.x**
```
# Get Login Password
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <ECR-REPOSITORY-URI>
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 180789647333.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-kubenginx

# Push the Docker Image
docker push <ECR-REPOSITORY-URI>:<TAG>
docker push 180789647333.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-kubenginx:1.0.0
```
- Verify the newly pushed docker image on AWS ECR. 
- Verify the vulnerability scan results. 

## Step-07: Using ECR Image with Amazon EKS

### Review the k8s manifests
- Understand the Deployment and Service kubernetes manifests present in folder **10-ECR-Elastic-Container-Registry\02-kube-manifests**
  - **Deployment:** 01-ECR-Nginx-Deployment.yml
  - **NodePort Service:** 02-ECR-Nginx-NodePortService.yml
  - **ALB Ingress Service:** 03-ECR-Nginx-ALB-IngressService.yml

### Verify ECR Access to EKS Worker Nodes
- Go to Services -> EC2 -> Running Instances > Select a Worker Node -> Description Tab
- Click on value in `IAM Role` field
```
# Sample Role Name 
eksctl-eksdemo1-nodegroup-eksdemo-NodeInstanceRole-1U4PSS3YLALN6
```
- In IAM on that specific role, verify **permissions** tab
- Policy with name `AmazonEC2ContainerRegistryReadOnly, AmazonEC2ContainerRegistryPowerUser` should be associated

### Deploy the kubernetes manifests
```
# Deploy
kubectl apply -f 02-kube-manifests/

# Verify
kubectl get deploy
kubectl get svc
kubectl get po
kubectl get ingress
```
### Access Application
- Wait for ALB Ingress to be provisioned
- Verify Route 53 DNS registration `ecrdemo.kubeoncloud.com`
```
# Get external ip of EKS Cluster Kubernetes worker nodes
kubectl get nodes -o wide

# Access Application
http://ecrdemo.kubeoncloud.com/index.html
```

## Step-08: Clean Up 
```
# Clean-Up
kubectl delete -f 02-kube-manifests/
```======================================================================
***********read the file 10-ECR-Elastic-Container-Registry-and-EKS/README.md ************
# AWS ECR - Elastic Container Registry Integration & EKS

## Step-01: What are we going to learn?
- We are going build a Docker image 
- Push to ECR Repository
- Update that ECR Image Repository URL in our Kubernetes Deployment manifest
- Deploy to EKS
- Kubernetes Deployment, NodePort Service, Ingress Service and External-DNS will be used to depict a full-on deployment
- We will access the ECR Demo Application using registered dns `http://ecrdemo.kubeoncloud.com`

## Step-02: ECR Terminology
 - **Registry:** An  ECR registry is provided to each AWS account; we can create image repositories in our registry and store images in them. 
- **Repository:** An ECR image repository contains our Docker images. 
- **Repository policy:** We can control access to our repositories and the images within them with repository policies. 
- **Authorization token:** Our Docker client must authenticate to Amazon ECR registries as an AWS user before it can push and pull images. The AWS CLI get-login command provides us with authentication credentials to pass to Docker. 
- **Image:** We can push and pull container images to our repositories.  

## Step-03: Pre-requisites
- Install required CLI software on your local desktop
   - **Install AWS CLI V2 version**
      - We have taken care of this step as part of [01-EKS-Create-Clusters-using-eksctl](/01-EKS-Create-Cluster-using-eksctl/01-01-Install-CLIs/README.md)
      - Documentation Reference: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html
   - **Install Docker CLI** 
      - We have taken of Docker local desktop installation as part of [Docker Fundamentals](https://github.com/stacksimplify/docker-fundamentals/tree/master/02-Docker-Installation) section 
      - Docker Desktop for MAC: https://docs.docker.com/docker-for-mac/install/
      - Docker Desktop for Windows: https://docs.docker.com/docker-for-windows/install/
      - Docker on Linux: https://docs.docker.com/install/linux/docker-ce/centos/

   - **On AWS Console**
      - We have taken care of this step as part of [01-EKS-Create-Clusters-using-eksctl](/01-EKS-Create-Cluster-using-eksctl/01-01-Install-CLIs/README.md)
      - Create Authorization Token for admin user if not created
      - **Configure AWS CLI with Authorization Token**
```
aws configure
AWS Access Key ID: ****
AWS Secret Access Key: ****
Default Region Name: us-east-1
```   

## Step-04: Create ECR Repository
- Create simple ECR repository via AWS Console 
   - Repository Name: aws-ecr-kubenginx
   - Tag Immutability: Enable
   - Scan on Push: Enable
- Explore ECR console. 
- **Create ECR Repository using AWS CLI**
```
aws ecr create-repository --repository-name aws-ecr-kubenginx --region us-east-1
aws ecr create-repository --repository-name <your-repo-name> --region <your-region>
```

## Step-05: Create Docker Image locally
- Navigate to folder **10-ECR-Elastic-Container-Registry\01-aws-ecr-kubenginx** from course github content download. 
- Create docker image locally
- Run it locally and test
```
# Build Docker Image
docker build -t <ECR-REPOSITORY-URI>:<TAG> . 
docker build -t 180789647333.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-kubenginx:1.0.0 . 

# Run Docker Image locally & Test
docker run --name <name-of-container> -p 80:80 --rm -d <ECR-REPOSITORY-URI>:<TAG>
docker run --name aws-ecr-kubenginx -p 80:80 --rm -d 180789647333.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-kubenginx:1.0.0

# Access Application locally
http://localhost

# Stop Docker Container
docker ps
docker stop aws-ecr-kubenginx
docker ps -a -q
```

## Step-06: Push Docker Image to AWS ECR
- Firstly, login to ECR Repository
- Push the docker image to ECR
- **AWS CLI Version 2.x**
```
# Get Login Password
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <ECR-REPOSITORY-URI>
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 180789647333.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-kubenginx

# Push the Docker Image
docker push <ECR-REPOSITORY-URI>:<TAG>
docker push 180789647333.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-kubenginx:1.0.0
```
- Verify the newly pushed docker image on AWS ECR. 
- Verify the vulnerability scan results. 

## Step-07: Using ECR Image with Amazon EKS

### Review the k8s manifests
- Understand the Deployment and Service kubernetes manifests present in folder **10-ECR-Elastic-Container-Registry\02-kube-manifests**
  - **Deployment:** 01-ECR-Nginx-Deployment.yml
  - **NodePort Service:** 02-ECR-Nginx-NodePortService.yml
  - **ALB Ingress Service:** 03-ECR-Nginx-ALB-IngressService.yml

### Verify ECR Access to EKS Worker Nodes
- Go to Services -> EC2 -> Running Instances > Select a Worker Node -> Description Tab
- Click on value in `IAM Role` field
```
# Sample Role Name 
eksctl-eksdemo1-nodegroup-eksdemo-NodeInstanceRole-1U4PSS3YLALN6
```
- In IAM on that specific role, verify **permissions** tab
- Policy with name `AmazonEC2ContainerRegistryReadOnly, AmazonEC2ContainerRegistryPowerUser` should be associated

### Deploy the kubernetes manifests
```
# Deploy
kubectl apply -f 02-kube-manifests/

# Verify
kubectl get deploy
kubectl get svc
kubectl get po
kubectl get ingress
```
### Access Application
- Wait for ALB Ingress to be provisioned
- Verify Route 53 DNS registration `ecrdemo.kubeoncloud.com`
```
# Get external ip of EKS Cluster Kubernetes worker nodes
kubectl get nodes -o wide

# Access Application
http://ecrdemo.kubeoncloud.com/index.html
```

## Step-08: Clean Up 
```
# Clean-Up
kubectl delete -f 02-kube-manifests/
```======================================================================
***********read the file 12-Microservices-Deployment-on-EKS/README.md ************
# Microservices Deployment on EKS

## Step-00: What are Microservices?
- Understand what are microservices on  a very high level

## Step-01: What are we going to learn in this section?
- We are going to deploy two microservices.
    - User Management Service
    - Notification Service

### Usecase Description
- User Management **Create User API**  will call Notification service **Send Notification API** to send an email to user when we create a user. 


### List of Docker Images used in this section
| Application Name                 | Docker Image Name                          |
| ------------------------------- | --------------------------------------------- |
| User Management Microservice | stacksimplify/kube-usermanagement-microservice:1.0.0 |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:1.0.0 |
| Notifications Microservice V2 | stacksimplify/kube-notifications-microservice:2.0.0 |

## Step-02: Pre-requisite -1: AWS RDS Database, ALB Ingress Controller & External DNS

### AWS RDS Database
- We have created AWS RDS Database as part of section [06-EKS-Storage-with-RDS-Database](/06-EKS-Storage-with-RDS-Database/README.md)
- We even created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database. 

### ALB Ingress Controller & External DNS
- We are going to deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Which means we should have both related pods running in our EKS cluster. 
- We have installed **ALB Ingress Controller** as part of section [08-01-ALB-Ingress-Install](/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install/README.md)
- We have installed **External DNS** as part of section [08-06-01-Deploy-ExternalDNS-on-EKS](/08-ELB-Application-LoadBalancers/08-06-ALB-Ingress-ExternalDNS/08-06-01-Deploy-ExternalDNS-on-EKS/README.md)
```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns pod running in default namespace
kubectl get pods
```


## Step-03: Pre-requisite-2: Create Simple Email Service - SES SMTP Credentials
### SMTP Credentials
- Go to Services -> Simple Email Service
- SMTP Settings --> Create My SMTP Credentials
- **IAM User Name:** append the default generated name with microservice or something so we have a reference of this IAM user created for our ECS Microservice deployment
- Download the credentials and update the same for below environment variables which you are going to provide in kubernetes manifest `04-NotificationMicroservice-Deployment.yml`
```
AWS_MAIL_SERVER_HOST=email-smtp.us-east-1.amazonaws.com
AWS_MAIL_SERVER_USERNAME=****
AWS_MAIL_SERVER_PASSWORD=***
AWS_MAIL_SERVER_FROM_ADDRESS= use-a-valid-email@gmail.com 
```
- **Important Note:** Environment variable AWS_MAIL_SERVER_FROM_ADDRESS value should be a **valid** email address and also verified in SES. 

### Verfiy Email Addresses to which notifications we need to send.
- We need two email addresses for testing Notification Service.  
-  **Email Addresses**
    - Verify a New Email Address
    - Email Address Verification Request will be sent to that address, click on link to verify your email. 
    - **From Address:** stacksimplify@gmail.com (replace with your ids during verification)
    - **To Address:** dkalyanreddy@gmail.com (replace with your ids during verification)
- **Important Note:** We need to ensure all the emails (FromAddress email) and (ToAddress emails) to be verified here. 
    - Reference Link: https://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html    
- Environment Variables
    - AWS_MAIL_SERVER_HOST=email-smtp.us-east-1.amazonaws.com
    - AWS_MAIL_SERVER_USERNAME=*****
    - AWS_MAIL_SERVER_PASSWORD=*****
    - AWS_MAIL_SERVER_FROM_ADDRESS=stacksimplify@gmail.com


## Step-04: Create Notification Microservice Deployment Manifest
- Update environment Variables for Notification Microservice
- **Notification Microservice Deployment**
```yml
          - name: AWS_MAIL_SERVER_HOST
            value: "smtp-service"
          - name: AWS_MAIL_SERVER_USERNAME
            value: "AKIABCDEDFASUBKLDOAX"
          - name: AWS_MAIL_SERVER_PASSWORD
            value: "Bdsdsadsd32qcsads65B4oLo7kMgmKZqhJtEipuE5unLx"
          - name: AWS_MAIL_SERVER_FROM_ADDRESS
            value: "stacksimplify@gmail.com"
```

## Step-05: Create Notification Microservice SMTP ExternalName Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: smtp-service
spec:
  type: ExternalName
  externalName: email-smtp.us-east-1.amazonaws.com
```

## Step-06: Create Notification Microservice NodePort Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: notification-clusterip-service
  labels:
    app: notification-restapp
spec:
  type: ClusterIP
  selector:
    app: notification-restapp
  ports:
  - port: 8096
    targetPort: 8096
```
## Step-07: Update User Management Microservice Deployment Manifest with Notification Service Environment Variables. 
- User Management Service new environment varibales related to Notification Microservice in addition to already which were configured related to MySQL
- Update in `02-UserManagementMicroservice-Deployment.yml`
```yml
          - name: NOTIFICATION_SERVICE_HOST
            value: "notification-clusterip-service"
          - name: NOTIFICATION_SERVICE_PORT
            value: "8096"    
```
## Step-08: Update ALB Ingress Service Kubernetes Manifest
- Update Ingress Service to ensure only target it is going to have is User Management Service
- Remove /app1, /app2 contexts
```yml
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: services.kubeoncloud.com, ums.kubeoncloud.com
spec:
  rules:
    - http:
        paths:
          - path: /* # SSL Redirect Setting
            backend:
              serviceName: ssl-redirect
              servicePort: use-annotation                   
          - path: /*
            backend:
              serviceName: usermgmt-restapp-nodeport-service
              servicePort: 8095              
```

## Step-09: Deploy Microservices manifests
```
# Deploy Microservices manifests
kubectl apply -f V1-Microservices/
```

## Step-10: Verify the Deployment using kubectl
```
# List Pods
kubectl get pods

# User Management Microservice Logs
kubectl logs -f $(kubectl get po | egrep -o 'usermgmt-microservice-[A-Za-z0-9-]+')

# Notification Microservice Logs
kubectl logs -f $(kubectl get po | egrep -o 'notification-microservice-[A-Za-z0-9-]+')

# External DNS Logs
kubectl logs -f $(kubectl get po | egrep -o 'external-dns-[A-Za-z0-9-]+')

# List Ingress
kubectl get ingress
```

## Step-11: Verify Microservices health-status via browser
```
# User Management Service Health-Status
https://services.kubeoncloud.com/usermgmt/health-status

# Notification Microservice Health-Status via User Management
https://services.kubeoncloud.com/usermgmt/notification-health-status
https://services.kubeoncloud.com/usermgmt/notification-service-info
```

## Step-12: Import postman project to Postman client on our desktop. 
- Import postman project
- Add environment url 
    - https://services.kubeoncloud.com (**Replace with your ALB DNS registered url on your environment**)

## Step-13: Test both Microservices using Postman
### User Management Service
- **Create User**
    - Verify the email id to confirm account creation email received.
- **List User**   
    - Verify if newly created user got listed. 
    


## Step-14: Rollout New Deployment - Set Image Option
```
# Rollout New Deployment using Set Image
kubectl set image deployment/notification-microservice notification-service=stacksimplify/kube-notifications-microservice:2.0.0 --record=true

# Verify Rollout Status
kubectl rollout status deployment/notification-microservice

# Verify ReplicaSets
kubectl get rs

# Verify Rollout History
kubectl rollout history deployment/notification-microservice

# Access Application (Should see V2)
https://services.kubeoncloud.com/usermgmt/notification-health-status

# Roll back to Previous Version
kubectl rollout undo deployment/notification-microservice

# Access Application (Should see V1)
https://services.kubeoncloud.com/usermgmt/notification-health-status
```    

## Step-15: Rollout New Deployment - kubectl Edit
```
# Rollout New Deployment using kubectl edit, change image version to 2.0.0
kubectl edit deployment/notification-microservice

# Verify Rollout Status
kubectl rollout status deployment/notification-microservice

# Verify ReplicaSets
kubectl get rs

# Verify Rollout History
kubectl rollout history deployment/notification-microservice

# Access Application (Should see V2)
https://services.kubeoncloud.com/usermgmt/notification-health-status

# Roll back to Previous Version
kubectl rollout undo deployment/notification-microservice

# Access Application (Should see V1)
https://services.kubeoncloud.com/usermgmt/notification-health-status
```

## Step-16: Rollout New Deployment - Update manifest & kubectl apply
```
# Rollout New Deployment by updating yaml manifest 2.0.0
kubectl apply -f kube-manifests/

# Verify Rollout Status
kubectl rollout status deployment/notification-microservice

# Verify ReplicaSets
kubectl get rs

# Verify Rollout History
kubectl rollout history deployment/notification-microservice

# Access Application (Should see V2)
https://services.kubeoncloud.com/usermgmt/notification-health-status

# Roll back to Previous Version
kubectl rollout undo deployment/notification-microservice

# Access Application (Should see V1)
https://services.kubeoncloud.com/usermgmt/notification-health-status
```

## Step-17: Clean-up
```
kubectl delete -f kube-manifests/    
```
======================================================================
***********read the file 12-Microservices-Deployment-on-EKS/README.md ************
# Microservices Deployment on EKS

## Step-00: What are Microservices?
- Understand what are microservices on  a very high level

## Step-01: What are we going to learn in this section?
- We are going to deploy two microservices.
    - User Management Service
    - Notification Service

### Usecase Description
- User Management **Create User API**  will call Notification service **Send Notification API** to send an email to user when we create a user. 


### List of Docker Images used in this section
| Application Name                 | Docker Image Name                          |
| ------------------------------- | --------------------------------------------- |
| User Management Microservice | stacksimplify/kube-usermanagement-microservice:1.0.0 |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:1.0.0 |
| Notifications Microservice V2 | stacksimplify/kube-notifications-microservice:2.0.0 |

## Step-02: Pre-requisite -1: AWS RDS Database, ALB Ingress Controller & External DNS

### AWS RDS Database
- We have created AWS RDS Database as part of section [06-EKS-Storage-with-RDS-Database](/06-EKS-Storage-with-RDS-Database/README.md)
- We even created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database. 

### ALB Ingress Controller & External DNS
- We are going to deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Which means we should have both related pods running in our EKS cluster. 
- We have installed **ALB Ingress Controller** as part of section [08-01-ALB-Ingress-Install](/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install/README.md)
- We have installed **External DNS** as part of section [08-06-01-Deploy-ExternalDNS-on-EKS](/08-ELB-Application-LoadBalancers/08-06-ALB-Ingress-ExternalDNS/08-06-01-Deploy-ExternalDNS-on-EKS/README.md)
```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns pod running in default namespace
kubectl get pods
```


## Step-03: Pre-requisite-2: Create Simple Email Service - SES SMTP Credentials
### SMTP Credentials
- Go to Services -> Simple Email Service
- SMTP Settings --> Create My SMTP Credentials
- **IAM User Name:** append the default generated name with microservice or something so we have a reference of this IAM user created for our ECS Microservice deployment
- Download the credentials and update the same for below environment variables which you are going to provide in kubernetes manifest `04-NotificationMicroservice-Deployment.yml`
```
AWS_MAIL_SERVER_HOST=email-smtp.us-east-1.amazonaws.com
AWS_MAIL_SERVER_USERNAME=****
AWS_MAIL_SERVER_PASSWORD=***
AWS_MAIL_SERVER_FROM_ADDRESS= use-a-valid-email@gmail.com 
```
- **Important Note:** Environment variable AWS_MAIL_SERVER_FROM_ADDRESS value should be a **valid** email address and also verified in SES. 

### Verfiy Email Addresses to which notifications we need to send.
- We need two email addresses for testing Notification Service.  
-  **Email Addresses**
    - Verify a New Email Address
    - Email Address Verification Request will be sent to that address, click on link to verify your email. 
    - **From Address:** stacksimplify@gmail.com (replace with your ids during verification)
    - **To Address:** dkalyanreddy@gmail.com (replace with your ids during verification)
- **Important Note:** We need to ensure all the emails (FromAddress email) and (ToAddress emails) to be verified here. 
    - Reference Link: https://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html    
- Environment Variables
    - AWS_MAIL_SERVER_HOST=email-smtp.us-east-1.amazonaws.com
    - AWS_MAIL_SERVER_USERNAME=*****
    - AWS_MAIL_SERVER_PASSWORD=*****
    - AWS_MAIL_SERVER_FROM_ADDRESS=stacksimplify@gmail.com


## Step-04: Create Notification Microservice Deployment Manifest
- Update environment Variables for Notification Microservice
- **Notification Microservice Deployment**
```yml
          - name: AWS_MAIL_SERVER_HOST
            value: "smtp-service"
          - name: AWS_MAIL_SERVER_USERNAME
            value: "AKIABCDEDFASUBKLDOAX"
          - name: AWS_MAIL_SERVER_PASSWORD
            value: "Bdsdsadsd32qcsads65B4oLo7kMgmKZqhJtEipuE5unLx"
          - name: AWS_MAIL_SERVER_FROM_ADDRESS
            value: "stacksimplify@gmail.com"
```

## Step-05: Create Notification Microservice SMTP ExternalName Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: smtp-service
spec:
  type: ExternalName
  externalName: email-smtp.us-east-1.amazonaws.com
```

## Step-06: Create Notification Microservice NodePort Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: notification-clusterip-service
  labels:
    app: notification-restapp
spec:
  type: ClusterIP
  selector:
    app: notification-restapp
  ports:
  - port: 8096
    targetPort: 8096
```
## Step-07: Update User Management Microservice Deployment Manifest with Notification Service Environment Variables. 
- User Management Service new environment varibales related to Notification Microservice in addition to already which were configured related to MySQL
- Update in `02-UserManagementMicroservice-Deployment.yml`
```yml
          - name: NOTIFICATION_SERVICE_HOST
            value: "notification-clusterip-service"
          - name: NOTIFICATION_SERVICE_PORT
            value: "8096"    
```
## Step-08: Update ALB Ingress Service Kubernetes Manifest
- Update Ingress Service to ensure only target it is going to have is User Management Service
- Remove /app1, /app2 contexts
```yml
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: services.kubeoncloud.com, ums.kubeoncloud.com
spec:
  rules:
    - http:
        paths:
          - path: /* # SSL Redirect Setting
            backend:
              serviceName: ssl-redirect
              servicePort: use-annotation                   
          - path: /*
            backend:
              serviceName: usermgmt-restapp-nodeport-service
              servicePort: 8095              
```

## Step-09: Deploy Microservices manifests
```
# Deploy Microservices manifests
kubectl apply -f V1-Microservices/
```

## Step-10: Verify the Deployment using kubectl
```
# List Pods
kubectl get pods

# User Management Microservice Logs
kubectl logs -f $(kubectl get po | egrep -o 'usermgmt-microservice-[A-Za-z0-9-]+')

# Notification Microservice Logs
kubectl logs -f $(kubectl get po | egrep -o 'notification-microservice-[A-Za-z0-9-]+')

# External DNS Logs
kubectl logs -f $(kubectl get po | egrep -o 'external-dns-[A-Za-z0-9-]+')

# List Ingress
kubectl get ingress
```

## Step-11: Verify Microservices health-status via browser
```
# User Management Service Health-Status
https://services.kubeoncloud.com/usermgmt/health-status

# Notification Microservice Health-Status via User Management
https://services.kubeoncloud.com/usermgmt/notification-health-status
https://services.kubeoncloud.com/usermgmt/notification-service-info
```

## Step-12: Import postman project to Postman client on our desktop. 
- Import postman project
- Add environment url 
    - https://services.kubeoncloud.com (**Replace with your ALB DNS registered url on your environment**)

## Step-13: Test both Microservices using Postman
### User Management Service
- **Create User**
    - Verify the email id to confirm account creation email received.
- **List User**   
    - Verify if newly created user got listed. 
    


## Step-14: Rollout New Deployment - Set Image Option
```
# Rollout New Deployment using Set Image
kubectl set image deployment/notification-microservice notification-service=stacksimplify/kube-notifications-microservice:2.0.0 --record=true

# Verify Rollout Status
kubectl rollout status deployment/notification-microservice

# Verify ReplicaSets
kubectl get rs

# Verify Rollout History
kubectl rollout history deployment/notification-microservice

# Access Application (Should see V2)
https://services.kubeoncloud.com/usermgmt/notification-health-status

# Roll back to Previous Version
kubectl rollout undo deployment/notification-microservice

# Access Application (Should see V1)
https://services.kubeoncloud.com/usermgmt/notification-health-status
```    

## Step-15: Rollout New Deployment - kubectl Edit
```
# Rollout New Deployment using kubectl edit, change image version to 2.0.0
kubectl edit deployment/notification-microservice

# Verify Rollout Status
kubectl rollout status deployment/notification-microservice

# Verify ReplicaSets
kubectl get rs

# Verify Rollout History
kubectl rollout history deployment/notification-microservice

# Access Application (Should see V2)
https://services.kubeoncloud.com/usermgmt/notification-health-status

# Roll back to Previous Version
kubectl rollout undo deployment/notification-microservice

# Access Application (Should see V1)
https://services.kubeoncloud.com/usermgmt/notification-health-status
```

## Step-16: Rollout New Deployment - Update manifest & kubectl apply
```
# Rollout New Deployment by updating yaml manifest 2.0.0
kubectl apply -f kube-manifests/

# Verify Rollout Status
kubectl rollout status deployment/notification-microservice

# Verify ReplicaSets
kubectl get rs

# Verify Rollout History
kubectl rollout history deployment/notification-microservice

# Access Application (Should see V2)
https://services.kubeoncloud.com/usermgmt/notification-health-status

# Roll back to Previous Version
kubectl rollout undo deployment/notification-microservice

# Access Application (Should see V1)
https://services.kubeoncloud.com/usermgmt/notification-health-status
```

## Step-17: Clean-up
```
kubectl delete -f kube-manifests/    
```
======================================================================
***********read the file 12-Microservices-Deployment-on-EKS/README.md ************
# Microservices Deployment on EKS

## Step-00: What are Microservices?
- Understand what are microservices on  a very high level

## Step-01: What are we going to learn in this section?
- We are going to deploy two microservices.
    - User Management Service
    - Notification Service

### Usecase Description
- User Management **Create User API**  will call Notification service **Send Notification API** to send an email to user when we create a user. 


### List of Docker Images used in this section
| Application Name                 | Docker Image Name                          |
| ------------------------------- | --------------------------------------------- |
| User Management Microservice | stacksimplify/kube-usermanagement-microservice:1.0.0 |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:1.0.0 |
| Notifications Microservice V2 | stacksimplify/kube-notifications-microservice:2.0.0 |

## Step-02: Pre-requisite -1: AWS RDS Database, ALB Ingress Controller & External DNS

### AWS RDS Database
- We have created AWS RDS Database as part of section [06-EKS-Storage-with-RDS-Database](/06-EKS-Storage-with-RDS-Database/README.md)
- We even created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database. 

### ALB Ingress Controller & External DNS
- We are going to deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Which means we should have both related pods running in our EKS cluster. 
- We have installed **ALB Ingress Controller** as part of section [08-01-ALB-Ingress-Install](/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install/README.md)
- We have installed **External DNS** as part of section [08-06-01-Deploy-ExternalDNS-on-EKS](/08-ELB-Application-LoadBalancers/08-06-ALB-Ingress-ExternalDNS/08-06-01-Deploy-ExternalDNS-on-EKS/README.md)
```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns pod running in default namespace
kubectl get pods
```


## Step-03: Pre-requisite-2: Create Simple Email Service - SES SMTP Credentials
### SMTP Credentials
- Go to Services -> Simple Email Service
- SMTP Settings --> Create My SMTP Credentials
- **IAM User Name:** append the default generated name with microservice or something so we have a reference of this IAM user created for our ECS Microservice deployment
- Download the credentials and update the same for below environment variables which you are going to provide in kubernetes manifest `04-NotificationMicroservice-Deployment.yml`
```
AWS_MAIL_SERVER_HOST=email-smtp.us-east-1.amazonaws.com
AWS_MAIL_SERVER_USERNAME=****
AWS_MAIL_SERVER_PASSWORD=***
AWS_MAIL_SERVER_FROM_ADDRESS= use-a-valid-email@gmail.com 
```
- **Important Note:** Environment variable AWS_MAIL_SERVER_FROM_ADDRESS value should be a **valid** email address and also verified in SES. 

### Verfiy Email Addresses to which notifications we need to send.
- We need two email addresses for testing Notification Service.  
-  **Email Addresses**
    - Verify a New Email Address
    - Email Address Verification Request will be sent to that address, click on link to verify your email. 
    - **From Address:** stacksimplify@gmail.com (replace with your ids during verification)
    - **To Address:** dkalyanreddy@gmail.com (replace with your ids during verification)
- **Important Note:** We need to ensure all the emails (FromAddress email) and (ToAddress emails) to be verified here. 
    - Reference Link: https://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html    
- Environment Variables
    - AWS_MAIL_SERVER_HOST=email-smtp.us-east-1.amazonaws.com
    - AWS_MAIL_SERVER_USERNAME=*****
    - AWS_MAIL_SERVER_PASSWORD=*****
    - AWS_MAIL_SERVER_FROM_ADDRESS=stacksimplify@gmail.com


## Step-04: Create Notification Microservice Deployment Manifest
- Update environment Variables for Notification Microservice
- **Notification Microservice Deployment**
```yml
          - name: AWS_MAIL_SERVER_HOST
            value: "smtp-service"
          - name: AWS_MAIL_SERVER_USERNAME
            value: "AKIABCDEDFASUBKLDOAX"
          - name: AWS_MAIL_SERVER_PASSWORD
            value: "Bdsdsadsd32qcsads65B4oLo7kMgmKZqhJtEipuE5unLx"
          - name: AWS_MAIL_SERVER_FROM_ADDRESS
            value: "stacksimplify@gmail.com"
```

## Step-05: Create Notification Microservice SMTP ExternalName Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: smtp-service
spec:
  type: ExternalName
  externalName: email-smtp.us-east-1.amazonaws.com
```

## Step-06: Create Notification Microservice NodePort Service
```yml
apiVersion: v1
kind: Service
metadata:
  name: notification-clusterip-service
  labels:
    app: notification-restapp
spec:
  type: ClusterIP
  selector:
    app: notification-restapp
  ports:
  - port: 8096
    targetPort: 8096
```
## Step-07: Update User Management Microservice Deployment Manifest with Notification Service Environment Variables. 
- User Management Service new environment varibales related to Notification Microservice in addition to already which were configured related to MySQL
- Update in `02-UserManagementMicroservice-Deployment.yml`
```yml
          - name: NOTIFICATION_SERVICE_HOST
            value: "notification-clusterip-service"
          - name: NOTIFICATION_SERVICE_PORT
            value: "8096"    
```
## Step-08: Update ALB Ingress Service Kubernetes Manifest
- Update Ingress Service to ensure only target it is going to have is User Management Service
- Remove /app1, /app2 contexts
```yml
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: services.kubeoncloud.com, ums.kubeoncloud.com
spec:
  rules:
    - http:
        paths:
          - path: /* # SSL Redirect Setting
            backend:
              serviceName: ssl-redirect
              servicePort: use-annotation                   
          - path: /*
            backend:
              serviceName: usermgmt-restapp-nodeport-service
              servicePort: 8095              
```

## Step-09: Deploy Microservices manifests
```
# Deploy Microservices manifests
kubectl apply -f V1-Microservices/
```

## Step-10: Verify the Deployment using kubectl
```
# List Pods
kubectl get pods

# User Management Microservice Logs
kubectl logs -f $(kubectl get po | egrep -o 'usermgmt-microservice-[A-Za-z0-9-]+')

# Notification Microservice Logs
kubectl logs -f $(kubectl get po | egrep -o 'notification-microservice-[A-Za-z0-9-]+')

# External DNS Logs
kubectl logs -f $(kubectl get po | egrep -o 'external-dns-[A-Za-z0-9-]+')

# List Ingress
kubectl get ingress
```

## Step-11: Verify Microservices health-status via browser
```
# User Management Service Health-Status
https://services.kubeoncloud.com/usermgmt/health-status

# Notification Microservice Health-Status via User Management
https://services.kubeoncloud.com/usermgmt/notification-health-status
https://services.kubeoncloud.com/usermgmt/notification-service-info
```

## Step-12: Import postman project to Postman client on our desktop. 
- Import postman project
- Add environment url 
    - https://services.kubeoncloud.com (**Replace with your ALB DNS registered url on your environment**)

## Step-13: Test both Microservices using Postman
### User Management Service
- **Create User**
    - Verify the email id to confirm account creation email received.
- **List User**   
    - Verify if newly created user got listed. 
    


## Step-14: Rollout New Deployment - Set Image Option
```
# Rollout New Deployment using Set Image
kubectl set image deployment/notification-microservice notification-service=stacksimplify/kube-notifications-microservice:2.0.0 --record=true

# Verify Rollout Status
kubectl rollout status deployment/notification-microservice

# Verify ReplicaSets
kubectl get rs

# Verify Rollout History
kubectl rollout history deployment/notification-microservice

# Access Application (Should see V2)
https://services.kubeoncloud.com/usermgmt/notification-health-status

# Roll back to Previous Version
kubectl rollout undo deployment/notification-microservice

# Access Application (Should see V1)
https://services.kubeoncloud.com/usermgmt/notification-health-status
```    

## Step-15: Rollout New Deployment - kubectl Edit
```
# Rollout New Deployment using kubectl edit, change image version to 2.0.0
kubectl edit deployment/notification-microservice

# Verify Rollout Status
kubectl rollout status deployment/notification-microservice

# Verify ReplicaSets
kubectl get rs

# Verify Rollout History
kubectl rollout history deployment/notification-microservice

# Access Application (Should see V2)
https://services.kubeoncloud.com/usermgmt/notification-health-status

# Roll back to Previous Version
kubectl rollout undo deployment/notification-microservice

# Access Application (Should see V1)
https://services.kubeoncloud.com/usermgmt/notification-health-status
```

## Step-16: Rollout New Deployment - Update manifest & kubectl apply
```
# Rollout New Deployment by updating yaml manifest 2.0.0
kubectl apply -f kube-manifests/

# Verify Rollout Status
kubectl rollout status deployment/notification-microservice

# Verify ReplicaSets
kubectl get rs

# Verify Rollout History
kubectl rollout history deployment/notification-microservice

# Access Application (Should see V2)
https://services.kubeoncloud.com/usermgmt/notification-health-status

# Roll back to Previous Version
kubectl rollout undo deployment/notification-microservice

# Access Application (Should see V1)
https://services.kubeoncloud.com/usermgmt/notification-health-status
```

## Step-17: Clean-up
```
kubectl delete -f kube-manifests/    
```
======================================================================
***********read the file 13-Microservices-Distributed-Tracing-using-AWS-XRay-on-EKS/README.md ************
# Microservices Distributed Tracing with X-Ray on AWS EKS

## Step-01: Introduction
### Introduction to AWS XRay & k8s DaemonSets 
- Understand about AWS X-Ray Services
- Understand Kubernetes DaemonSets
- Understand the AWS X-Ray and Microservices network design on EKS Cluster
- Understand about Service Map, Traces and Segments in AWS X-Ray

### Usecase Description
- User Management **getNotificationAppInfo**  will call Notification service **notification-xray** which will evetually send traces to AWS X-Ray service
- We are going to depict one Microservice calling other Microservice

### List of Docker Images used in this section
| Application Name                 | Docker Image Name                          |
| ------------------------------- | --------------------------------------------- |
| User Management Microservice | stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay |

## Step-02: Pre-requisite: AWS RDS Database, ALB Ingress Controller & External DNS

### AWS RDS Database
- We have created AWS RDS Database as part of section [06-EKS-Storage-with-RDS-Database](/06-EKS-Storage-with-RDS-Database/README.md)
- We even created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database. 

### ALB Ingress Controller & External DNS
- We are going to deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Which means we should have both related pods running in our EKS cluster. 
- We have installed **ALB Ingress Controller** as part of section [08-01-ALB-Ingress-Install](/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install/README.md)
- We have installed **External DNS** as part of section [08-06-01-Deploy-ExternalDNS-on-EKS](/08-ELB-Application-LoadBalancers/08-06-ALB-Ingress-ExternalDNS/08-06-01-Deploy-ExternalDNS-on-EKS/README.md)
```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns pod running in default namespace
kubectl get pods
```

## Step-03: Create IAM permissions for AWS X-Ray daemon
```
# Template
eksctl create iamserviceaccount \
    --name service_account_name \
    --namespace service_account_namespace \
    --cluster cluster_name \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess \
    --approve \
    --override-existing-serviceaccounts

# Replace Name, Namespace, Cluster Info (if any changes)
eksctl create iamserviceaccount \
    --name xray-daemon \
    --namespace default \
    --cluster eksdemo1 \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess \
    --approve \
    --override-existing-serviceaccounts
```

### Verify Service Account and AWS IAM Role
```
# List k8s Service Accounts
kubectl get sa

# Describe Service Account (Verify IAM Role annotated)
kubectl describe sa xray-daemon

# List IAM Roles on eksdemo1 Cluster created with eksctl
eksctl  get iamserviceaccount --cluster eksdemo1
```

## Step-04: Update IAM Role ARN in xray-k8s-daemonset.yml
### Get AWS IAM Role ARN for xray-daemon
```
# Get AWS IAM Role ARN
eksctl  get iamserviceaccount xray-daemon --cluster eksdemo1
```
### Update  xray-k8s-daemonset.yml
- File Name: kube-manifests/01-XRay-DaemonSet/xray-k8s-daemonset.yml
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: xray-daemon
  name: xray-daemon
  namespace: default
  # Update IAM Role ARN created for X-Ray access
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::180789647333:role/eksctl-eksdemo1-addon-iamserviceaccount-defa-Role1-20F5AWU2J61F
```

### Deploy X-Ray DaemonSet on our EKS Cluster
```
# Deploy
kubectl apply -f kube-manifests/01-XRay-DaemonSet/xray-k8s-daemonset.yml

# Verify Deployment, Service & Pod
kubectl get deploy,svc,pod

# Verify X-Ray Logs
kubectl logs -f <X-Ray Pod Name>
kubectl logs -f xray-daemon-phszp  

# List & Describe DaemonSet
kubectl get daemonset
kubectl describe daemonset xray-daemon
```

## Step-05: Review Microservices Application Deployment Manifests
- **02-UserManagementMicroservice-Deployment.yml**
```yml
# Change-1: Image Tag is 3.0.0-AWS-XRay-MySQLDB
      containers:
        - name: usermgmt-restapp
          image: stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME 
              value: "User-Management-Microservice"                
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"    
            - name: AWS_XRAY_CONTEXT_MISSING 
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured                                            
```
- **04-NotificationMicroservice-Deployment.yml**
```yml
# Change-1: Image Tag is 3.0.0-AWS-XRay
    spec:
      containers:
        - name: notification-service
          image: stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME 
              value: "V1-Notification-Microservice"              
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"      
            - name: AWS_XRAY_CONTEXT_MISSING 
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured                                            

```

## Step-06: Review Ingress Manifest
- **07-ALB-Ingress-SSL-Redirect-ExternalDNS.yml**
```yml
# Change-1-For-You: Update with your SSL Cert ARN when using template
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:180789647333:certificate/9f042b5d-86fd-4fad-96d0-c81c5abc71e1

# Change-2-For-You: Update with your "yourdomainname.com"
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: services-xray.kubeoncloud.com, xraydemo.kubeoncloud.com             
```

## Step-07: Deploy Manifests
```
# Deploy
kubectl apply -f kube-manifests/02-Applications

# Verify
kubectl get pods
```

## Step-08: Test
```
# Test
https://xraydemo.kubeoncloud.com/usermgmt/notification-xray
https://xraydemo.kubeoncloud.com/usermgmt/notification-xray

# Your Domain Name
https://<Replace-your-domain-name>/usermgmt/notification-xray
```

## Step-09: Clean-Up
- We are going to delete applications created as part of this section
- We are going to leave the xray daemon set running which we will leverage in our next section canary deployments in Kubernetes on EKS. 
```
# Delete Apps
kubectl delete -f kube-manifests/02-Applications
```

## References
- https://github.com/aws-samples/aws-xray-kubernetes/
- https://github.com/aws-samples/aws-xray-kubernetes/blob/master/xray-daemon/xray-k8s-daemonset.yaml
- https://aws.amazon.com/blogs/compute/application-tracing-on-kubernetes-with-aws-x-ray/
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html#xray-sdk-java-configuration-plugins
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-httpclients.html
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-filters.html
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-sqlclients.html
======================================================================
***********read the file 13-Microservices-Distributed-Tracing-using-AWS-XRay-on-EKS/README.md ************
# Microservices Distributed Tracing with X-Ray on AWS EKS

## Step-01: Introduction
### Introduction to AWS XRay & k8s DaemonSets 
- Understand about AWS X-Ray Services
- Understand Kubernetes DaemonSets
- Understand the AWS X-Ray and Microservices network design on EKS Cluster
- Understand about Service Map, Traces and Segments in AWS X-Ray

### Usecase Description
- User Management **getNotificationAppInfo**  will call Notification service **notification-xray** which will evetually send traces to AWS X-Ray service
- We are going to depict one Microservice calling other Microservice

### List of Docker Images used in this section
| Application Name                 | Docker Image Name                          |
| ------------------------------- | --------------------------------------------- |
| User Management Microservice | stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay |

## Step-02: Pre-requisite: AWS RDS Database, ALB Ingress Controller & External DNS

### AWS RDS Database
- We have created AWS RDS Database as part of section [06-EKS-Storage-with-RDS-Database](/06-EKS-Storage-with-RDS-Database/README.md)
- We even created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database. 

### ALB Ingress Controller & External DNS
- We are going to deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Which means we should have both related pods running in our EKS cluster. 
- We have installed **ALB Ingress Controller** as part of section [08-01-ALB-Ingress-Install](/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install/README.md)
- We have installed **External DNS** as part of section [08-06-01-Deploy-ExternalDNS-on-EKS](/08-ELB-Application-LoadBalancers/08-06-ALB-Ingress-ExternalDNS/08-06-01-Deploy-ExternalDNS-on-EKS/README.md)
```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns pod running in default namespace
kubectl get pods
```

## Step-03: Create IAM permissions for AWS X-Ray daemon
```
# Template
eksctl create iamserviceaccount \
    --name service_account_name \
    --namespace service_account_namespace \
    --cluster cluster_name \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess \
    --approve \
    --override-existing-serviceaccounts

# Replace Name, Namespace, Cluster Info (if any changes)
eksctl create iamserviceaccount \
    --name xray-daemon \
    --namespace default \
    --cluster eksdemo1 \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess \
    --approve \
    --override-existing-serviceaccounts
```

### Verify Service Account and AWS IAM Role
```
# List k8s Service Accounts
kubectl get sa

# Describe Service Account (Verify IAM Role annotated)
kubectl describe sa xray-daemon

# List IAM Roles on eksdemo1 Cluster created with eksctl
eksctl  get iamserviceaccount --cluster eksdemo1
```

## Step-04: Update IAM Role ARN in xray-k8s-daemonset.yml
### Get AWS IAM Role ARN for xray-daemon
```
# Get AWS IAM Role ARN
eksctl  get iamserviceaccount xray-daemon --cluster eksdemo1
```
### Update  xray-k8s-daemonset.yml
- File Name: kube-manifests/01-XRay-DaemonSet/xray-k8s-daemonset.yml
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: xray-daemon
  name: xray-daemon
  namespace: default
  # Update IAM Role ARN created for X-Ray access
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::180789647333:role/eksctl-eksdemo1-addon-iamserviceaccount-defa-Role1-20F5AWU2J61F
```

### Deploy X-Ray DaemonSet on our EKS Cluster
```
# Deploy
kubectl apply -f kube-manifests/01-XRay-DaemonSet/xray-k8s-daemonset.yml

# Verify Deployment, Service & Pod
kubectl get deploy,svc,pod

# Verify X-Ray Logs
kubectl logs -f <X-Ray Pod Name>
kubectl logs -f xray-daemon-phszp  

# List & Describe DaemonSet
kubectl get daemonset
kubectl describe daemonset xray-daemon
```

## Step-05: Review Microservices Application Deployment Manifests
- **02-UserManagementMicroservice-Deployment.yml**
```yml
# Change-1: Image Tag is 3.0.0-AWS-XRay-MySQLDB
      containers:
        - name: usermgmt-restapp
          image: stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME 
              value: "User-Management-Microservice"                
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"    
            - name: AWS_XRAY_CONTEXT_MISSING 
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured                                            
```
- **04-NotificationMicroservice-Deployment.yml**
```yml
# Change-1: Image Tag is 3.0.0-AWS-XRay
    spec:
      containers:
        - name: notification-service
          image: stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME 
              value: "V1-Notification-Microservice"              
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"      
            - name: AWS_XRAY_CONTEXT_MISSING 
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured                                            

```

## Step-06: Review Ingress Manifest
- **07-ALB-Ingress-SSL-Redirect-ExternalDNS.yml**
```yml
# Change-1-For-You: Update with your SSL Cert ARN when using template
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:180789647333:certificate/9f042b5d-86fd-4fad-96d0-c81c5abc71e1

# Change-2-For-You: Update with your "yourdomainname.com"
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: services-xray.kubeoncloud.com, xraydemo.kubeoncloud.com             
```

## Step-07: Deploy Manifests
```
# Deploy
kubectl apply -f kube-manifests/02-Applications

# Verify
kubectl get pods
```

## Step-08: Test
```
# Test
https://xraydemo.kubeoncloud.com/usermgmt/notification-xray
https://xraydemo.kubeoncloud.com/usermgmt/notification-xray

# Your Domain Name
https://<Replace-your-domain-name>/usermgmt/notification-xray
```

## Step-09: Clean-Up
- We are going to delete applications created as part of this section
- We are going to leave the xray daemon set running which we will leverage in our next section canary deployments in Kubernetes on EKS. 
```
# Delete Apps
kubectl delete -f kube-manifests/02-Applications
```

## References
- https://github.com/aws-samples/aws-xray-kubernetes/
- https://github.com/aws-samples/aws-xray-kubernetes/blob/master/xray-daemon/xray-k8s-daemonset.yaml
- https://aws.amazon.com/blogs/compute/application-tracing-on-kubernetes-with-aws-x-ray/
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html#xray-sdk-java-configuration-plugins
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-httpclients.html
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-filters.html
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-sqlclients.html
======================================================================
***********read the file 13-Microservices-Distributed-Tracing-using-AWS-XRay-on-EKS/README.md ************
# Microservices Distributed Tracing with X-Ray on AWS EKS

## Step-01: Introduction
### Introduction to AWS XRay & k8s DaemonSets 
- Understand about AWS X-Ray Services
- Understand Kubernetes DaemonSets
- Understand the AWS X-Ray and Microservices network design on EKS Cluster
- Understand about Service Map, Traces and Segments in AWS X-Ray

### Usecase Description
- User Management **getNotificationAppInfo**  will call Notification service **notification-xray** which will evetually send traces to AWS X-Ray service
- We are going to depict one Microservice calling other Microservice

### List of Docker Images used in this section
| Application Name                 | Docker Image Name                          |
| ------------------------------- | --------------------------------------------- |
| User Management Microservice | stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay |

## Step-02: Pre-requisite: AWS RDS Database, ALB Ingress Controller & External DNS

### AWS RDS Database
- We have created AWS RDS Database as part of section [06-EKS-Storage-with-RDS-Database](/06-EKS-Storage-with-RDS-Database/README.md)
- We even created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database. 

### ALB Ingress Controller & External DNS
- We are going to deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Which means we should have both related pods running in our EKS cluster. 
- We have installed **ALB Ingress Controller** as part of section [08-01-ALB-Ingress-Install](/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install/README.md)
- We have installed **External DNS** as part of section [08-06-01-Deploy-ExternalDNS-on-EKS](/08-ELB-Application-LoadBalancers/08-06-ALB-Ingress-ExternalDNS/08-06-01-Deploy-ExternalDNS-on-EKS/README.md)
```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns pod running in default namespace
kubectl get pods
```

## Step-03: Create IAM permissions for AWS X-Ray daemon
```
# Template
eksctl create iamserviceaccount \
    --name service_account_name \
    --namespace service_account_namespace \
    --cluster cluster_name \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess \
    --approve \
    --override-existing-serviceaccounts

# Replace Name, Namespace, Cluster Info (if any changes)
eksctl create iamserviceaccount \
    --name xray-daemon \
    --namespace default \
    --cluster eksdemo1 \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess \
    --approve \
    --override-existing-serviceaccounts
```

### Verify Service Account and AWS IAM Role
```
# List k8s Service Accounts
kubectl get sa

# Describe Service Account (Verify IAM Role annotated)
kubectl describe sa xray-daemon

# List IAM Roles on eksdemo1 Cluster created with eksctl
eksctl  get iamserviceaccount --cluster eksdemo1
```

## Step-04: Update IAM Role ARN in xray-k8s-daemonset.yml
### Get AWS IAM Role ARN for xray-daemon
```
# Get AWS IAM Role ARN
eksctl  get iamserviceaccount xray-daemon --cluster eksdemo1
```
### Update  xray-k8s-daemonset.yml
- File Name: kube-manifests/01-XRay-DaemonSet/xray-k8s-daemonset.yml
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: xray-daemon
  name: xray-daemon
  namespace: default
  # Update IAM Role ARN created for X-Ray access
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::180789647333:role/eksctl-eksdemo1-addon-iamserviceaccount-defa-Role1-20F5AWU2J61F
```

### Deploy X-Ray DaemonSet on our EKS Cluster
```
# Deploy
kubectl apply -f kube-manifests/01-XRay-DaemonSet/xray-k8s-daemonset.yml

# Verify Deployment, Service & Pod
kubectl get deploy,svc,pod

# Verify X-Ray Logs
kubectl logs -f <X-Ray Pod Name>
kubectl logs -f xray-daemon-phszp  

# List & Describe DaemonSet
kubectl get daemonset
kubectl describe daemonset xray-daemon
```

## Step-05: Review Microservices Application Deployment Manifests
- **02-UserManagementMicroservice-Deployment.yml**
```yml
# Change-1: Image Tag is 3.0.0-AWS-XRay-MySQLDB
      containers:
        - name: usermgmt-restapp
          image: stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME 
              value: "User-Management-Microservice"                
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"    
            - name: AWS_XRAY_CONTEXT_MISSING 
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured                                            
```
- **04-NotificationMicroservice-Deployment.yml**
```yml
# Change-1: Image Tag is 3.0.0-AWS-XRay
    spec:
      containers:
        - name: notification-service
          image: stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME 
              value: "V1-Notification-Microservice"              
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"      
            - name: AWS_XRAY_CONTEXT_MISSING 
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured                                            

```

## Step-06: Review Ingress Manifest
- **07-ALB-Ingress-SSL-Redirect-ExternalDNS.yml**
```yml
# Change-1-For-You: Update with your SSL Cert ARN when using template
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:180789647333:certificate/9f042b5d-86fd-4fad-96d0-c81c5abc71e1

# Change-2-For-You: Update with your "yourdomainname.com"
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: services-xray.kubeoncloud.com, xraydemo.kubeoncloud.com             
```

## Step-07: Deploy Manifests
```
# Deploy
kubectl apply -f kube-manifests/02-Applications

# Verify
kubectl get pods
```

## Step-08: Test
```
# Test
https://xraydemo.kubeoncloud.com/usermgmt/notification-xray
https://xraydemo.kubeoncloud.com/usermgmt/notification-xray

# Your Domain Name
https://<Replace-your-domain-name>/usermgmt/notification-xray
```

## Step-09: Clean-Up
- We are going to delete applications created as part of this section
- We are going to leave the xray daemon set running which we will leverage in our next section canary deployments in Kubernetes on EKS. 
```
# Delete Apps
kubectl delete -f kube-manifests/02-Applications
```

## References
- https://github.com/aws-samples/aws-xray-kubernetes/
- https://github.com/aws-samples/aws-xray-kubernetes/blob/master/xray-daemon/xray-k8s-daemonset.yaml
- https://aws.amazon.com/blogs/compute/application-tracing-on-kubernetes-with-aws-x-ray/
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-configuration.html#xray-sdk-java-configuration-plugins
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-httpclients.html
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-filters.html
- https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-sqlclients.html
======================================================================
***********read the file 14-Microservices-Canary-Deployments/README.md ************
# Microservices Canary Deployments 

## Step-01: Introduction
### Usecase Description
- User Management **getNotificationAppInfo**  will call Notification service V1 and V2 versions.
- We will distribute traffic between V1 and V2 versions of Notification service as per our choice based on Replicas

| NS V1 Replicas | NS V2 Replicas | Traffic Distribution | 
| -------------- | -------------- | -------------------- |
| 4 | 0 | 100% traffic to NS V1 Version |
| 3 | 1 | 25% traffic to NS V2 Version |
| 2 | 2 | 50% traffic to NS V2 Version |
| 1 | 3 | 75% traffic to NS V2 Version |
| 0 | 4 | 100% traffic to NS V2 Version |

- In our demo, we are going to distribute 50% traffic to each version (V1 and V2). 
- NS V1 - 2 replicas and NS V2 - 2 replicas
- We are going to depict one Microservice calling other Microservices with different versions in AWS X-Ray

### List of Docker Images used in this section
| Application Name                 | Docker Image Name                          |
| ------------------------------- | --------------------------------------------- |
| User Management Microservice | stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay |
| Notifications Microservice V2 | stacksimplify/kube-notifications-microservice:4.0.0-AWS-XRay |

## Step-02: Pre-requisite: AWS RDS Database, ALB Ingress Controller, External DNS & X-Ray Daemon

### AWS RDS Database
- We have created AWS RDS Database as part of section [06-EKS-Storage-with-RDS-Database](/06-EKS-Storage-with-RDS-Database/README.md)
- We even created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database. 

### ALB Ingress Controller & External DNS
- We are going to deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Which means we should have both related pods running in our EKS cluster. 
- We have installed **ALB Ingress Controller** as part of section [08-01-ALB-Ingress-Install](/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install/README.md)
- We have installed **External DNS** as part of section [08-06-01-Deploy-ExternalDNS-on-EKS](/08-ELB-Application-LoadBalancers/08-06-ALB-Ingress-ExternalDNS/08-06-01-Deploy-ExternalDNS-on-EKS/README.md)

### XRay Daemon
- We are going to view the application traces in AWS X-Ray.
- We need XRay Daemon running as Daemonset for that. 
```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns & xray-daemon pod running in default namespace
kubectl get pods
```

## Step-03: Review Deployment Manifest for V2 Notification Service
- We are going to distribute 50% traffic to each of the V1 and V2 version of application


| NS V1 Replicas | NS V2 Replicas | Traffic Distribution | 
| -------------- | -------------- | -------------------- |
| 2 | 2 | 50% traffic to NS V2 Version |

- **08-V2-NotificationMicroservice-Deployment.yml**
```yml
# Change-1: Image Tag is 4.0.0-AWS-XRay
    spec:
      containers:
        - name: notification-service
          image: stacksimplify/kube-notifications-microservice:4.0.0-AWS-XRay

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME 
              value: "V2-Notification-Microservice"              
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"      
            - name: AWS_XRAY_CONTEXT_MISSING 
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured    
```


## Step-04: Review Ingress Manifest
- **07-ALB-Ingress-SSL-Redirect-ExternalDNS.yml**
```yml
# Change-1-For-You: Update with your SSL Cert ARN when using template
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:180789647333:certificate/9f042b5d-86fd-4fad-96d0-c81c5abc71e1

# Change-2-For-You: Update with your "yourdomainname.com"
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: canarydemo.kubeoncloud.com
```

## Step-05: Deploy Manifests
```
# Deploy
kubectl apply -f kube-manifests/

# Verify
kubectl get deploy,svc,pod
```
## Step-06: Test
```
# Test
https://canarydemo.kubeoncloud.com/usermgmt/notification-xray

# Your Domain Name
https://<Replace-your-domain-name>/usermgmt/notification-xray
```

## Step-07: What is happening in the background?
- As far as `Notification Cluster IP Service` selector label matches to `Notificaiton V1 and V2 Deployment manifests selector.matchLables`  those respective pods are picked to send traffic.
```yml
# Notification Cluster IP Service - Selector Label
  selector:
    app: notification-restapp

# Notification V1 and V2 Deployment - Selector Match Labels
  selector:
    matchLabels:
      app: notification-restapp         
```

## Step-08: Clean-Up
- We are going to delete applications created as part of this section
```
# Delete Apps
kubectl delete -f kube-manifests/
```

## Step-09: Downside of this approach
- We will review the downside in a presentation

## Step-10: Best ways for Canary Deployments
- Istio (Open Source)
- AWS AppMesh (AWS version of Istio)======================================================================
***********read the file 14-Microservices-Canary-Deployments/README.md ************
# Microservices Canary Deployments 

## Step-01: Introduction
### Usecase Description
- User Management **getNotificationAppInfo**  will call Notification service V1 and V2 versions.
- We will distribute traffic between V1 and V2 versions of Notification service as per our choice based on Replicas

| NS V1 Replicas | NS V2 Replicas | Traffic Distribution | 
| -------------- | -------------- | -------------------- |
| 4 | 0 | 100% traffic to NS V1 Version |
| 3 | 1 | 25% traffic to NS V2 Version |
| 2 | 2 | 50% traffic to NS V2 Version |
| 1 | 3 | 75% traffic to NS V2 Version |
| 0 | 4 | 100% traffic to NS V2 Version |

- In our demo, we are going to distribute 50% traffic to each version (V1 and V2). 
- NS V1 - 2 replicas and NS V2 - 2 replicas
- We are going to depict one Microservice calling other Microservices with different versions in AWS X-Ray

### List of Docker Images used in this section
| Application Name                 | Docker Image Name                          |
| ------------------------------- | --------------------------------------------- |
| User Management Microservice | stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay |
| Notifications Microservice V2 | stacksimplify/kube-notifications-microservice:4.0.0-AWS-XRay |

## Step-02: Pre-requisite: AWS RDS Database, ALB Ingress Controller, External DNS & X-Ray Daemon

### AWS RDS Database
- We have created AWS RDS Database as part of section [06-EKS-Storage-with-RDS-Database](/06-EKS-Storage-with-RDS-Database/README.md)
- We even created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database. 

### ALB Ingress Controller & External DNS
- We are going to deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Which means we should have both related pods running in our EKS cluster. 
- We have installed **ALB Ingress Controller** as part of section [08-01-ALB-Ingress-Install](/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install/README.md)
- We have installed **External DNS** as part of section [08-06-01-Deploy-ExternalDNS-on-EKS](/08-ELB-Application-LoadBalancers/08-06-ALB-Ingress-ExternalDNS/08-06-01-Deploy-ExternalDNS-on-EKS/README.md)

### XRay Daemon
- We are going to view the application traces in AWS X-Ray.
- We need XRay Daemon running as Daemonset for that. 
```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns & xray-daemon pod running in default namespace
kubectl get pods
```

## Step-03: Review Deployment Manifest for V2 Notification Service
- We are going to distribute 50% traffic to each of the V1 and V2 version of application


| NS V1 Replicas | NS V2 Replicas | Traffic Distribution | 
| -------------- | -------------- | -------------------- |
| 2 | 2 | 50% traffic to NS V2 Version |

- **08-V2-NotificationMicroservice-Deployment.yml**
```yml
# Change-1: Image Tag is 4.0.0-AWS-XRay
    spec:
      containers:
        - name: notification-service
          image: stacksimplify/kube-notifications-microservice:4.0.0-AWS-XRay

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME 
              value: "V2-Notification-Microservice"              
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"      
            - name: AWS_XRAY_CONTEXT_MISSING 
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured    
```


## Step-04: Review Ingress Manifest
- **07-ALB-Ingress-SSL-Redirect-ExternalDNS.yml**
```yml
# Change-1-For-You: Update with your SSL Cert ARN when using template
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:180789647333:certificate/9f042b5d-86fd-4fad-96d0-c81c5abc71e1

# Change-2-For-You: Update with your "yourdomainname.com"
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: canarydemo.kubeoncloud.com
```

## Step-05: Deploy Manifests
```
# Deploy
kubectl apply -f kube-manifests/

# Verify
kubectl get deploy,svc,pod
```
## Step-06: Test
```
# Test
https://canarydemo.kubeoncloud.com/usermgmt/notification-xray

# Your Domain Name
https://<Replace-your-domain-name>/usermgmt/notification-xray
```

## Step-07: What is happening in the background?
- As far as `Notification Cluster IP Service` selector label matches to `Notificaiton V1 and V2 Deployment manifests selector.matchLables`  those respective pods are picked to send traffic.
```yml
# Notification Cluster IP Service - Selector Label
  selector:
    app: notification-restapp

# Notification V1 and V2 Deployment - Selector Match Labels
  selector:
    matchLabels:
      app: notification-restapp         
```

## Step-08: Clean-Up
- We are going to delete applications created as part of this section
```
# Delete Apps
kubectl delete -f kube-manifests/
```

## Step-09: Downside of this approach
- We will review the downside in a presentation

## Step-10: Best ways for Canary Deployments
- Istio (Open Source)
- AWS AppMesh (AWS version of Istio)======================================================================
***********read the file 14-Microservices-Canary-Deployments/README.md ************
# Microservices Canary Deployments 

## Step-01: Introduction
### Usecase Description
- User Management **getNotificationAppInfo**  will call Notification service V1 and V2 versions.
- We will distribute traffic between V1 and V2 versions of Notification service as per our choice based on Replicas

| NS V1 Replicas | NS V2 Replicas | Traffic Distribution | 
| -------------- | -------------- | -------------------- |
| 4 | 0 | 100% traffic to NS V1 Version |
| 3 | 1 | 25% traffic to NS V2 Version |
| 2 | 2 | 50% traffic to NS V2 Version |
| 1 | 3 | 75% traffic to NS V2 Version |
| 0 | 4 | 100% traffic to NS V2 Version |

- In our demo, we are going to distribute 50% traffic to each version (V1 and V2). 
- NS V1 - 2 replicas and NS V2 - 2 replicas
- We are going to depict one Microservice calling other Microservices with different versions in AWS X-Ray

### List of Docker Images used in this section
| Application Name                 | Docker Image Name                          |
| ------------------------------- | --------------------------------------------- |
| User Management Microservice | stacksimplify/kube-usermanagement-microservice:3.0.0-AWS-XRay-MySQLDB |
| Notifications Microservice V1 | stacksimplify/kube-notifications-microservice:3.0.0-AWS-XRay |
| Notifications Microservice V2 | stacksimplify/kube-notifications-microservice:4.0.0-AWS-XRay |

## Step-02: Pre-requisite: AWS RDS Database, ALB Ingress Controller, External DNS & X-Ray Daemon

### AWS RDS Database
- We have created AWS RDS Database as part of section [06-EKS-Storage-with-RDS-Database](/06-EKS-Storage-with-RDS-Database/README.md)
- We even created a `externalName service: 01-MySQL-externalName-Service.yml` in our Kubernetes manifests to point to that RDS Database. 

### ALB Ingress Controller & External DNS
- We are going to deploy a application which will also have a `ALB Ingress Service` and also will register its DNS name in Route53 using `External DNS`
- Which means we should have both related pods running in our EKS cluster. 
- We have installed **ALB Ingress Controller** as part of section [08-01-ALB-Ingress-Install](/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install/README.md)
- We have installed **External DNS** as part of section [08-06-01-Deploy-ExternalDNS-on-EKS](/08-ELB-Application-LoadBalancers/08-06-ALB-Ingress-ExternalDNS/08-06-01-Deploy-ExternalDNS-on-EKS/README.md)

### XRay Daemon
- We are going to view the application traces in AWS X-Ray.
- We need XRay Daemon running as Daemonset for that. 
```
# Verify alb-ingress-controller pod running in namespace kube-system
kubectl get pods -n kube-system

# Verify external-dns & xray-daemon pod running in default namespace
kubectl get pods
```

## Step-03: Review Deployment Manifest for V2 Notification Service
- We are going to distribute 50% traffic to each of the V1 and V2 version of application


| NS V1 Replicas | NS V2 Replicas | Traffic Distribution | 
| -------------- | -------------- | -------------------- |
| 2 | 2 | 50% traffic to NS V2 Version |

- **08-V2-NotificationMicroservice-Deployment.yml**
```yml
# Change-1: Image Tag is 4.0.0-AWS-XRay
    spec:
      containers:
        - name: notification-service
          image: stacksimplify/kube-notifications-microservice:4.0.0-AWS-XRay

# Change-2: New Environment Variables related to AWS X-Ray
            - name: AWS_XRAY_TRACING_NAME 
              value: "V2-Notification-Microservice"              
            - name: AWS_XRAY_DAEMON_ADDRESS
              value: "xray-service.default:2000"      
            - name: AWS_XRAY_CONTEXT_MISSING 
              value: "LOG_ERROR"  # Log an error and continue, Ideally RUNTIME_ERROR – Throw a runtime exception which is default option if not configured    
```


## Step-04: Review Ingress Manifest
- **07-ALB-Ingress-SSL-Redirect-ExternalDNS.yml**
```yml
# Change-1-For-You: Update with your SSL Cert ARN when using template
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:180789647333:certificate/9f042b5d-86fd-4fad-96d0-c81c5abc71e1

# Change-2-For-You: Update with your "yourdomainname.com"
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: canarydemo.kubeoncloud.com
```

## Step-05: Deploy Manifests
```
# Deploy
kubectl apply -f kube-manifests/

# Verify
kubectl get deploy,svc,pod
```
## Step-06: Test
```
# Test
https://canarydemo.kubeoncloud.com/usermgmt/notification-xray

# Your Domain Name
https://<Replace-your-domain-name>/usermgmt/notification-xray
```

## Step-07: What is happening in the background?
- As far as `Notification Cluster IP Service` selector label matches to `Notificaiton V1 and V2 Deployment manifests selector.matchLables`  those respective pods are picked to send traffic.
```yml
# Notification Cluster IP Service - Selector Label
  selector:
    app: notification-restapp

# Notification V1 and V2 Deployment - Selector Match Labels
  selector:
    matchLabels:
      app: notification-restapp         
```

## Step-08: Clean-Up
- We are going to delete applications created as part of this section
```
# Delete Apps
kubectl delete -f kube-manifests/
```

## Step-09: Downside of this approach
- We will review the downside in a presentation

## Step-10: Best ways for Canary Deployments
- Istio (Open Source)
- AWS AppMesh (AWS version of Istio)======================================================================
