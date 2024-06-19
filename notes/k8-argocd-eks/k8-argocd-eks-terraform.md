To create a private EKS cluster using Terraform, install Argo CD, and expose the Argo CD server using an Ingress controller, follow these steps:

### Step 1: Set Up Your AWS Environment

1. **Install Terraform**: Ensure you have Terraform installed. If not, follow the [Installation Guide](https://learn.hashicorp.com/tutorials/terraform/install-cli).

2. **Configure AWS CLI**:
   ```sh
   aws configure
   ```

### Step 2: Create a Private EKS Cluster with Terraform

1. **Create Terraform Configuration File**:
   Create a directory for your Terraform project and navigate into it:
   ```sh
   mkdir terraform-eks && cd terraform-eks
   ```

   Create a file named `main.tf` and add the following configuration:
   ```hcl
   provider "aws" {
     region = "us-west-2"
   }

   module "vpc" {
     source  = "terraform-aws-modules/vpc/aws"
     version = "3.10.0"

     name = "eks-vpc"
     cidr = "10.0.0.0/16"

     azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
     private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]

     enable_nat_gateway = true
     single_nat_gateway = true

     tags = {
       Name = "eks-vpc"
     }
   }

   module "eks" {
     source          = "terraform-aws-modules/eks/aws"
     version         = "18.0.0"
     cluster_name    = "private-cluster"
     cluster_version = "1.21"
     subnets         = module.vpc.private_subnets
     vpc_id          = module.vpc.vpc_id

     node_groups = {
       eks_nodes = {
         desired_capacity = 2
         max_capacity     = 3
         min_capacity     = 1

         instance_type = "t3.medium"

         key_name = "my-key-pair"
       }
     }

     enable_irsa = true

     tags = {
       Name = "eks-cluster"
     }
   }

   output "cluster_endpoint" {
     description = "EKS cluster endpoint"
     value       = module.eks.cluster_endpoint
   }

   output "cluster_security_group_id" {
     description = "EKS cluster security group id"
     value       = module.eks.cluster_security_group_id
   }

   output "node_security_group_id" {
     description = "EKS node security group id"
     value       = module.eks.node_security_group_id
   }
   ```

2. **Initialize Terraform**:
   ```sh
   terraform init
   ```

3. **Apply Terraform Configuration**:
   ```sh
   terraform apply
   ```

   Confirm the apply operation by typing `yes`.

### Step 3: Install Argo CD on the EKS Cluster

1. **Update kubeconfig**:
   ```sh
   aws eks --region us-west-2 update-kubeconfig --name private-cluster
   ```

2. **Create Namespace for Argo CD**:
   ```sh
   kubectl create namespace argocd
   ```

3. **Install Argo CD**:
   ```sh
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

4. **Install Argo CD CLI (Optional)**:
   ```sh
   VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | jq -r .tag_name)
   curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64
   chmod +x argocd
   sudo mv argocd /usr/local/bin/
   ```

### Step 4: Expose Argo CD Server Using Ingress

1. **Install NGINX Ingress Controller**:
   ```sh
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
   ```

2. **Create an Ingress Resource for Argo CD**:
   Create a file named `argocd-ingress.yaml`:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: argocd-server-ingress
     namespace: argocd
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     rules:
     - host: YOUR.DOMAIN.COM
       http:
         paths:
         - path: /
           pathType: ImplementationSpecific
           backend:
             service:
               name: argocd-server
               port:
                 number: 80
   ```

   Apply the Ingress resource:
   ```sh
   kubectl apply -f argocd-ingress.yaml
   ```

3. **Update DNS**: Point your domain (`YOUR.DOMAIN.COM`) to the external IP of the NGINX Ingress controller.

### Step 5: Access Argo CD UI

1. **Get Argo CD Initial Admin Password**:
   ```sh
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```

2. **Access the UI**: Open a web browser and navigate to `http://YOUR.DOMAIN.COM`. Log in with the username `admin` and the password retrieved in the previous step.

### Summary
You've created a private EKS cluster using Terraform, installed Argo CD, and exposed the Argo CD server using an Ingress controller. This setup ensures that your Argo CD UI is accessible via a custom domain with proper routing through the NGINX Ingress controller. Make sure to replace `YOUR.DOMAIN.COM` with your actual domain name.