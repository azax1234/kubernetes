Deploying in a cluster
---------------------

1. Create GKE cluster.
2. Connect to cloud shell
3. copy the gcp connect command from GKE to connect cloudshell to your GKE cluster.


gcloud container clusters get-credentials my-cluster --region us-east1 --project sonic-ivy-424616-s5

4. Run below commands to create and expose the cluster.

kubectl create deployment go-sample-web --image=azax1234/go-webapp-sample:0.0.1.RELEASE

kubectl expose deployment go-sample-web --type=LoadBalancer --port=8000
