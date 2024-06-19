Create a private GKE cluster, installed Argo CD, and exposed the Argo CD server using an Ingress controller. This setup ensures that your Argo CD UI is accessible via a custom domain with proper routing through the NGINX Ingress controller.

### Step 1: Set Up Your GCP Environment

1. **Open Cloud Shell**: Access Google Cloud Console and open Cloud Shell.
2. **Set Default Project**: Set your default project where you want to create the GKE cluster.
   ```sh
   gcloud config set project YOUR_PROJECT_ID
   ```

### Step 2: Create a Private GKE Cluster

1. **Enable Required APIs**:
   ```sh
   gcloud services enable container.googleapis.com compute.googleapis.com
   ```

2. **Create the GKE Cluster**:
   ```sh
   gcloud container clusters create my-argo-cluster \
       --zone us-central1-a \
       --enable-ip-alias \
       --enable-private-nodes \
       --master-ipv4-cidr 172.16.0.32/28 \
       --create-subnetwork name=my-subnet \
       --enable-master-authorized-networks \
       --master-authorized-networks 0.0.0.0/0 \
       --enable-private-endpoint
   ```

3. **Get Cluster Credentials**:
   ```sh
   gcloud container clusters get-credentials my-argo-cluster --zone us-central1-a
   ```

### Step 3: Install Argo CD

1. **Create Namespace for Argo CD**:
   ```sh
   kubectl create namespace argocd
   ```

2. **Install Argo CD**:
   ```sh
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Install Argo CD CLI (Optional)**:
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
You've created a private GKE cluster, installed Argo CD, and exposed the Argo CD server using an Ingress controller. This setup ensures that your Argo CD UI is accessible via a custom domain with proper routing through the NGINX Ingress controller.