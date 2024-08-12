## Commands

Dry run and generating yaml manifest file ( Uses local repo to pull image )

```sh
kubectl create deployment explorecalifornia.com --dry-run=client --image localhost:5000/explorecalifornia.com -o yaml > deployment.yaml
## dry run can be either server or client
## local repo isa t port 5000

kubectl get pods -l explorecalifornia.com 