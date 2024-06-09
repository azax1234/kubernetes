
### All pods habve unique IP

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get pods -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP            NODE                                  NOMINATED NODE   READINESS GATES
go-sample-web1-98d4b89c7-52lmd   1/1     Running   0          42m   10.54.0.134   gk3-my-cluster-pool-2-5c363fdd-cjrw   <none>           <none>
go-sample-web1-98d4b89c7-86sf2   1/1     Running   0          46m   10.54.0.130   gk3-my-cluster-pool-2-5c363fdd-cjrw   <none>           <none>
go-sample-web1-98d4b89c7-zhnct   1/1     Running   0          41m   10.54.0.95    gk3-my-cluster-pool-2-5c363fdd-lbsy   <none>           <none>

## each time a pod is deleted a new pod with new IP is created.
### however the service UI IP remains same as we have  a LB service exposed.

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl delete pod go-sample-web1-98d4b89c7-zhnct 
pod "go-sample-web1-98d4b89c7-zhnct" deleted
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get pods -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP            NODE                                  NOMINATED NODE   READINESS GATES
go-sample-web1-98d4b89c7-52lmd   1/1     Running   0          55m   10.54.0.134   gk3-my-cluster-pool-2-5c363fdd-cjrw   <none>           <none>
go-sample-web1-98d4b89c7-86sf2   1/1     Running   0          59m   10.54.0.130   gk3-my-cluster-pool-2-5c363fdd-cjrw   <none>           <none>
go-sample-web1-98d4b89c7-sqccn   1/1     Running   0          5s    10.54.0.96    gk3-my-cluster-pool-2-5c363fdd-lbsy   <none>           <none>

## Here we have a  GKE LB service create4d for us.
### The kubernetes service is a internal cluster service used by K8

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get services
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
go-sample-web1   LoadBalancer   34.118.239.249   34.74.58.252   8000:30716/TCP   8d
kubernetes       ClusterIP      34.118.224.1     <none>         443/TCP          8d

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get services -o wide
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE   SELECTOR
go-sample-web1   LoadBalancer   34.118.239.249   34.74.58.252   8000:30716/TCP   8d    app=go-sample-web1
kubernetes       ClusterIP      34.118.224.1     <none>         443/TCP          8d    <none>
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ 

## To watch services

ubuntu@ip-172-31-15-80:~$ kubectl get svc --watch
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
hello-world-rest-api   LoadBalancer   34.118.227.162   34.75.241.251   8080:30376/TCP   4h10m
kubernetes             ClusterIP      34.118.224.1     <none>          443/TCP          12d
