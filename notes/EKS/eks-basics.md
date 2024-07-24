
***create cluster***
```sh
eksctl create cluster --name demo-cluster --region ca-central-1
```

***update kubeconfig***
```sh
aws eks update-kubeconfig --name demo-cluster --region ca-central-1
```
***create fargateprofile to deploy app ina  new namespace as by default only kube-system  profile and default profile will be there. This is only to allow deployment in new namespace, you still have to create the new namspace via deployment file***
```sh
eksctl create fargateprofile --cluster demo-cluster --region ca-central-1 --name demo-app --namespace demo-namespace
```

kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml

$ kubectl get ingress -n game-2048
NAME           CLASS   HOSTS   ADDRESS   PORTS   AGE
ingress-2048   alb     *                 80      3m9s
[cloudshell-user@ip-10-132-38-231 ~]$ 

eksctl create iamserviceaccount \
--cluster=demo-cluster \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::073405128519:policy/AWSLoadBalancerControllerIAMPolicy \
--approve


kubectl edit deployment aws-load-balancer-controller -n kube-system


kubectl describe deployment -n kube-system aws-load-balancer-controller


kubectl get ingress -n game-2048
NAME           CLASS   HOSTS   ADDRESS                                                                      PORTS   AGE
ingress-2048   alb     *       k8s-game2048-ingress2-bcac0b5b37-1929766795.ca-central-1.elb.amazonaws.com   80      16h
------------------------
create cluster -- create fargate profile - create deployment ( deplymnet, service, ingress)- attach IAM policy to the oidc connector of the cluster - add IAM oilicy toa  service account - use that service account for ALB ingress controler)



=-----------

3 tier project
---------------



