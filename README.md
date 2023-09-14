# jfrog-k8s-integration
1. Used JFrog image to create pod and deployment on Kubernetes cluster
2. Used Kubernetes Docker secret to pull image from JFrog

### Prerqusites 
- Linux systesm
- AWS CLI, Kubectl, Eksctl, jf cli, AWS account, K8s cluster, JF artificatory

### High level steps 
[Follow Link for tools installation](https://sunitabachhav2007.hashnode.dev/prometheus-and-grafana-dashboard-on-eks-cluster-using-helm-chart#heading-setup-an-aws-ec2-instance)
- Creat Linux system on AWS cloud
- install required softwares
  - AWS CLI 
  - Kubectl
  - Eksctl
  - JF cli
  - Docker engine
 - Configure JFrog artificatory [For installation](https://jfrog.com/help/r/jfrog-installation-setup-documentation/installing-artifactory)
 - Upload artificats on JFrog
 - Create K8s cluster
 - Create K8s secret
 - Create Pod / Deployment using the secret and pull the image from JFrog

## Step By Step configuration
### Upload artficats in Jfrog
- create local Docker reposiroty on jfrog
  ![image](https://github.com/anand40090/jfrog-k8s-integration/assets/32446706/960dd4f5-786f-4b26-b396-b63fe4df21ec)
- Once reposiroty is created, go to artifacts and do the basic setup to generate user token, Push Image, Pull Image
  ![image](https://github.com/anand40090/jfrog-k8s-integration/assets/32446706/d98ccdc9-0706-4612-b8ea-b7b293ce7e9b)
- Download image from dockerhub or build the docker image locally on linux system using docker build command
  - Login with docker user on linux system ``` docker login --username foo --password-stdin ```
  - Download the image from docket hub ``` docker pull jfcloud.jfrog.io/docker-repo-1/sp:1 ```
      
- Push the docker image to JFrog from docker enabled system
 ```
  docker tag bc621bd8e41b jfcloud.jfrog.io/docker-repo-1/sp:1
  docker push jfcloud.jfrog.io/docker-repo-1/sp:1
  ```
  ![image](https://github.com/anand40090/jfrog-k8s-integration/assets/32446706/2f370b9e-c2a6-43b7-87aa-e6d7ab2185a9)
### Create Kubernetes cluster 
```
eksctl create cluster --name DC \
--node-type t2.medium \
--nodes 2 \
--nodes-min 2 \
--nodes-max 3 \
--region ap-south-1 \
--zones=ap-south-1a,ap-south-1b \
--authenticator-role-arn=arn:aws:iam::XXXXXXXXXXXXXXX:instance-profile/SSM-FullAccess \
--auto-kubeconfig \
--asg-access \
--external-dns-access \
--appmesh-access \
--alb-ingress-access
```
![image](https://github.com/anand40090/jfrog-k8s-integration/assets/32446706/fcb6da78-3b55-4562-8ebe-b5fb012882a5)

### Create the Kubernetes Secret for docker config 
 ``` Docker config Secrets – Stores the credentials for accessing a Docker registry for images ```

#### Define variables
```
SECRET_NAME="jfrog-secret"
REGISTRY_SERVER="your-jfrog-artifactory-url"
USERNAME="your-username"
PASSWORD="your-password"
EMAIL="your-email"
```
#### Create the Kubernetes Secret
```
kubectl create secret docker-registry $SECRET_NAME \
  --docker-server=$REGISTRY_SERVER \
  --docker-username=$USERNAME \
  --docker-password=$PASSWORD \
  --docker-email=$EMAIL
```
### Create Kubernetes deployment / pod using jfrog artifactory image (Use docker secret to pull image)
- Create pod
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo-2
spec:
  containers:
  - name: demo-container
    image: jfcloud.jfrog.io/docker-repo-1/sp:1
    envFrom:
    - secretRef:
       name: jfsecret
```
- Create Deployment
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo-2
spec:
  containers:
  - name: demo-container
    image: jfcloud.jfrog.io/docker-repo-1/sp:1
    envFrom:
    - secretRef:
       name: jfsecret
admin1@sonarqube-jfrog:~$ cat test1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test1
  name: test1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test1
    spec:
      containers:
      - image: jfcloud.jfrog.io/docker-repo-1/sp:1
        name: sp
        resources: {}
      imagePullSecrets:
      - name: jfsecret
status: {}
```
![image](https://github.com/anand40090/jfrog-k8s-integration/assets/32446706/64281155-bdc0-4a0f-a132-7bc39eb0dd65)

### Useful Links 
- [For Docker](https://docs.docker.com)
- [For JF cli](https://jfrog.com/help/r/jfrog-cli/jfrog-cli-plugins)
- [JF cli cheat sheet](https://media.jfrog.com/wp-content/uploads/2021/03/30185137/JFrogCLICheatSheet.pdf)
- [Kubernetes Secrets: How to Create, Use, and Manage Secrets in Kubernetes](https://www.youtube.com/watch?v=zi_8gccSsig&list=PLY63ZQr2Y5BHkJJhwPjJuJ41CIyv3m7Ru&index=23)
- [Security with Istio: Using Authorization Policies](https://youtu.be/no9kg4vzXUo?si=y6mmml2b1FbLxizf)
- [Basic Example of Spring Boot Microservice using Istio as Service Mesh](https://arupmishra91.medium.com/basic-example-of-spring-boot-microservice-using-istio-as-service-mesh-673073dcab07)


    
  
