# All Command for Devops Engineer

## Install node js via nvm
```bash
# installs nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
# download and install Node.js (you may need to restart the terminal)
nvm install 20
# verifies the right Node.js version is in the environment
node -v # should print `v20.17.0`
# verifies the right npm version is in the environment
npm -v # should print `10.8.2`
```
## Install Docker Engine
for linux (ubuntu)
```bash
sudo apt update
sudo apt install docker.io
usermod -aG docker $(whoami)
newgrp docker
systemctl restart docker
```
check docker version and make sure that the Docker is running
```bash
docker --version
docker run hello-world
```
login to docker hub
```bash
echo "<your-dockerhub-password>" | docker login --username <your-dockerhub-username> --password-stdin
```
Docker related commands
```bash
# build the docker image
docker build -t <your-dockerhub-username>/<image-name>:<tag> .

# Docker image run
docker run -d -p <host-port>:<container-port> -t <your-dockerhub-username>/<image-name>:<tag>

# push in docker hub
docker push <your-dockerhub-username>/<image-name>:<tag>

# Displays running containers:
docker ps

# docker ps -a
docker ps -a

# stop the docker running container
docker stop <container-id or container-name>

# Removes a stopped container:
docker rm <container-id or container-name>

# Deletes an image from the local system:
docker rmi <image-name>:<tag>

# Removes all Docker images:
docker rmi $(docker images -q)

# View details about a container (e.g., configuration, network settings):
docker inspect <container-id or container-name>
```
## 2. Install kubectl,eksctl,aws-cli

install aws cli

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
```

install eksctl

```bash
curl -s --location "https://github.com/weaveworks/eksctl/releases/download/v0.149.0/ek
sctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
install kubectl
```bash
# Download the latest version of kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
# Make the binary executable
chmod +x ./kubectl
# Move the binary to your PATH
sudo mv ./kubectl /usr/local/bin
# Verify kubectl installation
kubectl version --client
```
## 3. Create a new EKS cluster
Install a EKS cluster with EKSCTL
```bash
eksctl create cluster --name demo-cluster --region us-east-1
```
Delete the cluster
```bash
eksctl delete cluster --name demo-cluster --region us-east-1
```

## 4.configure aws load balancer in EKS cluster

Download the policy for the LoadBalancer prerequisite.

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

Create the IAM policy using the below command

```bash
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```

Create OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=Three-Tier-K8s-EKS-Cluster --approve
```

Create a Service Account by using below command and replace your account ID with your one

```bash
eksctl create iamserviceaccount --cluster=Three-Tier-K8s-EKS-Cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your_account_id>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-1
```

Run the below command to deploy the AWS Load Balancer Controller

```bash
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```

After 2 minutes, run the command below to check whether your pods are running or not.

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
## 5.Install Nginx Load Balancer Ingress Controller 

Deploy the below manifest

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```
## 6.Install Argo CD

Install Argo CD using manifests

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Access the Argo CD UI (Loadbalancer service)

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Access the Argo CD UI (Loadbalancer service) -For Windows

```bash
kubectl patch svc argocd-server -n argocd -p '{\"spec\": {\"type\": \"LoadBalancer\"}}'
```

Get the Loadbalancer service IP

```bash
kubectl get svc argocd-server -n argocd
```
