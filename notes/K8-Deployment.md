kubectl get rs
kubectl get rs -o wide
## Get deployment and container details
kubectl get deployments -o wide
NAME             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS         IMAGES                                    SELECTOR
go-sample-web1   3/3     3            3           8d    go-webapp-sample   azax1234/go-webapp-sample:0.0.1.RELEASE   app=go-sample-web1


# Set new image to the deployed container
## kubectl set image deployment deployment_name container_name=DUMMY:TEST
### Setting it to a  dummy image so that the deployment fails

kubectl set image deployment go-sample-web1 go-webapp-sample=DUMMY:TEST

# new pod creation will fail and service will still point to old deployment image
kubectl get rs -o wide
kubectl get pods
kubectl describe pod hello-world-rest-api-85995ddd5c-msjsm
kubectl get events --sort-by=.metadata.creationTimestamp

# When you set to a actual image, when the deployment is successful, old replicaset and pods will be dropped and new pods will be spinned up

kubectl set image deployment go-sample-web1 go-webapp-sample=azax1234/go-webapp-sample:0.0.2.RELEASE



-----
# Scenario
## I have 3 apps installed right now.

                                         9m48s        1       simple-webapp-mysql.17d715494014a9fb
ubuntu@ip-172-31-15-80:~$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
go-sample-web1-98d4b89c7-52lmd          1/1     Running   0          3d
go-sample-web1-98d4b89c7-86sf2          1/1     Running   0          3d
go-sample-web1-98d4b89c7-sqccn          1/1     Running   0          2d23h
hello-world-rest-api-65dbf8d4b7-72jkr   0/1     Pending   0          11h
hello-world-rest-api-7b6555f9b6-wh7cw   1/1     Running   0          21h
simple-webapp-mysql-86b455f456-4gplp    1/1     Running   0          21
#

### I want to delete 2 of these and keep only hello-world-rest-api
### So I delete the rest 2 by using their app label

buntu@ip-172-31-15-80:~$ kubectl delete all -l app=go-sample-web1
pod "go-sample-web1-98d4b89c7-52lmd" deleted
pod "go-sample-web1-98d4b89c7-86sf2" deleted
pod "go-sample-web1-98d4b89c7-sqccn" deleted
service "go-sample-web1" deleted
#
deployment.apps "go-sample-web1" deleted
replicaset.apps "go-sample-web1-54b855d657" deleted
replicaset.apps "go-sample-web1-698559d667" deleted
replicaset.apps "go-sample-web1-98d4b89c7" deleted

# I have only one now
## This is the 0.0.2.Realese
ubuntu@ip-172-31-15-80:~$ kubectl get deployments -o wide
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS             IMAGES                                       SELECTOR
hello-world-rest-api   1/1     1            1           22h   hello-world-rest-api   in28min/hello-world-rest-api:0.0.2.RELEASE   app=hello-world-rest-api

## If I have set the record=true option I will be able to see the history. Looks like while deploying the 0.0.2.Release I didnt add the record option.
ubuntu@ip-172-31-15-80:~$ kubectl rollout history deployment hello-world-rest-api
deployment.apps/hello-world-rest-api 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.3.RELEASE --record=true
3         kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.3.RELEASE --record=true
#

# Changing to 1st version now
ubuntu@ip-172-31-15-80:~$ kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.1.RELEASE --record=true
#
ubuntu@ip-172-31-15-80:~$ kubectl rollout history deployment hello-world-rest-api
deployment.apps/hello-world-rest-api 
REVISION  CHANGE-CAUSE
2         kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.3.RELEASE --record=true
3         kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.3.RELEASE --record=true
4         kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.1.RELEASE --record=true
## You can also roll back to specific revision from above by using `kubectl rollout undo deployment hello-world-rest-api --to-revision=3`
ubuntu@ip-172-31-15-80:~$ kubectl get deployments -o wide
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS             IMAGES                                       SELECTOR
hello-world-rest-api   1/1     1            1           22h   hello-world-rest-api   in28min/hello-world-rest-api:0.0.1.RELEASE   app=hello-world-rest-api
#
## New pod will take some time to spin up, till then app will be ponting to older version
ubuntu@ip-172-31-15-80:~$ kubectl get pods -o wide
NAME                                    READY   STATUS    RESTARTS   AGE     IP           NODE                                  NOMINATED NODE   READINESS GATES
hello-world-rest-api-5ffc8fdfb8-p6bdf   0/1     Pending   0          2m33s   <none>       <none>                                <none>           <none>
hello-world-rest-api-65dbf8d4b7-t8kl9   1/1     Running   0          6m28s   10.54.0.97   gk3-my-cluster-pool-2-5c363fdd-lbsy   <none>           <none>
#

#App now points to v1
ubuntu@ip-172-31-15-80:~$ kubectl get pods -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP            NODE                                  NOMINATED NODE   READINESS GATES
hello-world-rest-api-5ffc8fdfb8-n7b47   1/1     Running   0          13m   10.54.0.101   gk3-my-cluster-pool-2-5c363fdd-lbsy   <none>           <none>
# You can ceheck logs of pods
ubuntu@ip-172-31-15-80:~$ kubectl logs hello-world-rest-api-5ffc8fdfb8-n7b47
2024-06-08 17:11:14.781 DEBUG 1 --- [           main] .c.l.ClasspathLoggingApplicationListener 


# Now lets take a depp  ddive into depolyments using yaml file (K8 is descriptive)

`kubectl get deployment hello-world-rest-api -o yaml > deployment.yaml`


apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-world-rest-api
  name: hello-world-rest-api
  namespace: default
spec:
  replicas: 1 #You can update the replica set instance count
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: hello-world-rest-api
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: hello-world-rest-api
    spec:
      containers:
      - image: in28min/hello-world-rest-api:0.0.1.RELEASE
        imagePullPolicy: IfNotPresent #Pull from docker if not persent
        name: hello-world-rest-api
        resources:
          limits:
            cpu: 500m
            ephemeral-storage: 1Gi
            memory: 2Gi
          requests:
            cpu: 500m
            ephemeral-storage: 1Gi
            memory: 2Gi
        securityContext:
          capabilities:
            drop:
            - NET_RAW
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      terminationGracePeriodSeconds: 3

## I have update the replicaset to 2 and run below command to deploy the app again using deployment.yaml. The no of pods have iuncreased to 2 from 1.

buntu@ip-172-31-15-80:~$ kubectl apply -f deployment.yaml
Warning: resource deployments/hello-world-rest-api is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/hello-world-rest-api configured
ubuntu@ip-172-31-15-80:~$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-rest-api-5ffc8fdfb8-52cbt   0/1     Pending   0          10s
hello-world-rest-api-5ffc8fdfb8-n7b47   1/1     Running   0          47m
ubuntu@ip-172-31-15-80:~$ kubectl get rs
NAME                              DESIRED   CURRENT   READY   AGE
hello-world-rest-api-5ffc8fdfb8   2         2         1       22h

## You can add service details as well tothe deployment.yaml by adding --- in the 1st line at the end.
# Services.yaml

`kubectl get services hello-world-rest-api -o yaml > services.yaml`
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-world-rest-api
  name: hello-world-rest-api
  namespace: default
spec:
  allocateLoadBalancerNodePorts: true
  ports:
  - nodePort: 30376
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello-world-rest-api
  sessionAffinity: None #sticky user sessions
  type: LoadBalancer


#### I have added deployments and services into one file deployment.yaml (refer deployment.yaml) and going to deploy entire fleet with only the yaml.

## lets delete the entire fleet first
ubuntu@ip-172-31-15-80:~$ kubectl delete all -l app=hello-world-rest-api
pod "hello-world-rest-api-5ffc8fdfb8-52cbt" deleted
pod "hello-world-rest-api-5ffc8fdfb8-n7b47" deleted
service "hello-world-rest-api" deleted
deployment.apps "hello-world-rest-api" deleted
ubuntu@ip-172-31-15-80:~$ kubectl get all
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   34.118.224.1   <none>        443/TCP   12d

# deploying yaml now

ubuntu@ip-172-31-15-80:~$ kubectl apply -f deployment.yaml
deployment.apps/hello-world-rest-api created
service/hello-world-rest-api created
ubuntu@ip-172-31-15-80:~$ kubectl get all
NAME                                        READY   STATUS    RESTARTS   AGE
pod/hello-world-rest-api-6bf9896596-n4gvk   1/1     Running   0          30s

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/hello-world-rest-api   LoadBalancer   34.118.232.111   <pending>     8080:30376/TCP   30s
service/kubernetes             ClusterIP      34.118.224.1     <none>        443/TCP          12d

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-world-rest-api   1/1     1            1           30s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-world-rest-api-6bf9896596   1         1         1       30s

## Scaling replicaset to 3
ubuntu@ip-172-31-15-80:~$ kubectl scale deployment hello-world-rest-api --replicas=3
deployment.apps/hello-world-rest-api scaled
ubuntu@ip-172-31-15-80:~$ kubectl get rs
NAME                              DESIRED   CURRENT   READY   AGE
hello-world-rest-api-6bf9896596   3         3         1       3m16s

## I can also watch ther service from terminal using the watch command. It pings the service every 2 seconds

ubuntu@ip-172-31-15-80:~$ watch curl http://34.139.127.53:8080/hello-world
Every 2.0s: curl http://34.139.127.53:8080/hello-world                                                                                    ip-172-31-15-80: Sat Jun  8 18:29:41 2024
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0 100    21  100    21    0     0    308      0 --:--:-- --:--:-- --:--:--   308
Hello World  V1 n4gvk
# Since I have manually updated the relica set to 3 and the original deployment.yaml, I can saee the differnce in actual status like below

ubuntu@ip-172-31-15-80:~$ kubectl diff -f deployment.yaml 

-  replicas: 3
+  replicas: 1
   revisionHistoryLimit: 10
   selector:
     matchLabels:


## We can have 2 versions of the app deployed as well. Specify the version labels in the deployment.yaml in the selectors(refer depolyment-with-label.yaml in my notes)

buntu@ip-172-31-15-80:~$ kubectl apply -f deployment.yaml
deployment.apps/hello-world-rest-api-v1 created
deployment.apps/hello-world-rest-api-v2 created
service/hello-world-rest-api unchanged
ubuntu@ip-172-31-15-80:~$ watch curl http://34.75.241.251:8080/hello-world
ubuntu@ip-172-31-15-80:~$ kubectl get all
NAME                                           READY   STATUS    RESTARTS   AGE
pod/hello-world-rest-api-v1-b79b8bb55-fxxgl    1/1     Running   0          76s
pod/hello-world-rest-api-v2-75f95bc6b7-bkzwx   0/1     Pending   0          76s

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
service/hello-world-rest-api   LoadBalancer   34.118.227.162   34.75.241.251   8080:30376/TCP   6m26s
service/kubernetes             ClusterIP      34.118.224.1     <none>          443/TCP          12d

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-world-rest-api-v1   1/1     1            1           77s
deployment.apps/hello-world-rest-api-v2   0/1     1            0           76s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-world-rest-api-v1-b79b8bb55    1         1         1       76s
replicaset.apps/hello-world-rest-api-v2-75f95bc6b7   1         1         0       76s
ubuntu@ip-172-31-15-80:~$ kubectl get all
# This will create 2 versions of the app and sice the service is unchanged it will point to both by load balancing
#
---
ubuntu@ip-172-31-15-80:~$ watch curl http://34.75.241.251:8080/hello-world
#
Every 2.0s: curl http://34.75.241.251:8080/hello-world                                                                                              ip-172-31-15-80: Sat Jun  8 20:13:52 2024

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0 100    21  100    21    0     0    300      0 --:--:-- --:--:-- --:--:--   304
Hello World  V2 bkzwx



ubuntu@ip-172-31-15-80:~$ watch curl http://34.75.241.251:8080/hello-world
#
Every 2.0s: curl http://34.75.241.251:8080/hello-world                                                                                              ip-172-31-15-80: Sat Jun  8 20:14:07 2024

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0 100    21  100    21    0     0    308      0 --:--:-- --:--:-- --:--:--   313
Hello World  V1 fxxgl

### We can also trick the service to  point to only one versin of the app although both are deployed by adding a the selector label version to desired version
ubuntu@ip-172-31-15-80:~$ kubectl diff -f deployment.yaml
diff -u -N /tmp/LIVE-704226310/v1.Service.default.hello-world-rest-api /tmp/MERGED-413727659/v1.Service.default.hello-world-rest-api
--- /tmp/LIVE-704226310/v1.Service.default.hello-world-rest-api	2024-06-08 20:18:30.269103190 +0000
+++ /tmp/MERGED-413727659/v1.Service.default.hello-world-rest-api	2024-06-08 20:18:30.269103190 +0000
@@ -10,7 +10,6 @@
   - service.kubernetes.io/load-balancer-cleanup
   labels:
     app: hello-world-rest-api
-    version: v1
   name: hello-world-rest-api
   namespace: default
   resourceVersion: "14002840"
@@ -32,6 +31,7 @@
     targetPort: 8080
   selector:
     app: hello-world-rest-api
+    version: v2 ## version updated to v2 here to pint only to v2 of the app
   sessionAffinity: None
   type: LoadBalancer
 status:
ubuntu@ip-172-31-15-80:~$ kubectl apply -f deployment.yaml
deployment.apps/hello-world-rest-api-v1 unchanged
deployment.apps/hello-world-rest-api-v2 unchanged
service/hello-world-rest-api configured
## it will pont to v2 only

