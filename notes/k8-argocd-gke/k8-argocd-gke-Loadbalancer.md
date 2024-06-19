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
  ```sh
  kubectl edit service argocd-server -n argocd
  ```
### Step 5: Access the Argo CD UI with the exposed endpoints
```sh
  kubectl edit service argocd-server -n argocd
```

```sh
kubectl get services -n argocd
NAME                                      TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP      34.118.234.179   <none>          7000/TCP,8080/TCP            87m
argocd-dex-server                         ClusterIP      34.118.228.45    <none>          5556/TCP,5557/TCP,5558/TCP   87m
argocd-metrics                            ClusterIP      34.118.228.238   <none>          8082/TCP                     87m
argocd-notifications-controller-metrics   ClusterIP      34.118.227.59    <none>          9001/TCP                     87m
argocd-redis                              ClusterIP      34.118.233.155   <none>          6379/TCP                     87m
argocd-repo-server                        ClusterIP      34.118.229.141   <none>          8081/TCP,8084/TCP            87m
argocd-server                             LoadBalancer   34.118.239.131   35.188.215.80   80:31625/TCP,443:30520/TCP   87m
argocd-server-metrics                     ClusterIP      34.118.230.83    <none>          8083/TCP                     87m
```
***Access with 35.188.215.80***