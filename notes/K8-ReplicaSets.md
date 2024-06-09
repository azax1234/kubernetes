

ReplicaSets (They are like Autoscaling in AWS)
------------------------------------------------

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get replicasets
NAME                       DESIRED   CURRENT   READY   AGE
go-sample-web1-98d4b89c7   1         1         1       8d
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get rs
NAME                       DESIRED   CURRENT   READY   AGE
go-sample-web1-98d4b89c7   1         1         1       8d

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get pods 
NAME                             READY   STATUS    RESTARTS   AGE
go-sample-web1-98d4b89c7-tcjr5   1/1     Running   0          6d11h

### Only 1 pod is running as desired

###Lets delete that pod

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl delete pod go-sample-web1-98d4b89c7-tcjr5
pod "go-sample-web1-98d4b89c7-tcjr5" deleted
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get pods 
NAME                             READY   STATUS    RESTARTS   AGE
go-sample-web1-98d4b89c7-wdkwx   0/1     Pending   0          0s

### Note the pod name. Its differnt from te earlier one. Replicasets spin up one new pod as desired is 1

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get pods 
NAME                             READY   STATUS    RESTARTS   AGE
go-sample-web1-98d4b89c7-wdkwx   1/1     Running   0          10s
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get rs
NAME                       DESIRED   CURRENT   READY   AGE
go-sample-web1-98d4b89c7   1         1         1       8d

### Les see the tiemstamps od events
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get events --sort-by=.metadata.creationTimestamp
LAST SEEN   TYPE     REASON             OBJECT                                MESSAGE
2m30s       Normal   Killing            pod/go-sample-web1-98d4b89c7-tcjr5    Stopping container go-webapp-sample
2m26s       Normal   Scheduled          pod/go-sample-web1-98d4b89c7-wdkwx    Successfully assigned default/go-sample-web1-98d4b89c7-wdkwx to gk3-my-cluster-pool-2-5c363fdd-lbsy
2m26s       Normal   SuccessfulCreate   replicaset/go-sample-web1-98d4b89c7   Created pod: go-sample-web1-98d4b89c7-wdkwx
2m25s       Normal   Pulled             pod/go-sample-web1-98d4b89c7-wdkwx    Container image "azax1234/go-webapp-sample:0.0.1.RELEASE" already present on machine
2m25s       Normal   Created            pod/go-sample-web1-98d4b89c7-wdkwx    Created container go-webapp-sample
2m25s       Normal   Started            pod/go-sample-web1-98d4b89c7-wdkwx    Started container go-webapp-sample

### We can now try to scale our deployment to have 3 replicas

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
go-sample-web1   1/1     1            1           8d

azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl scale deployment go-sample-web1 --replicas=3
deployment.apps/go-sample-web1 scaled
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get events --sort-by=.metadata.creationTimestamp
LAST SEEN   TYPE      REASON              OBJECT                                MESSAGE
4m42s       Normal    Killing             pod/go-sample-web1-98d4b89c7-tcjr5    Stopping container go-webapp-sample
4m38s       Normal    Scheduled           pod/go-sample-web1-98d4b89c7-wdkwx    Successfully assigned default/go-sample-web1-98d4b89c7-wdkwx to gk3-my-cluster-pool-2-5c363fdd-lbsy
4m38s       Normal    SuccessfulCreate    replicaset/go-sample-web1-98d4b89c7   Created pod: go-sample-web1-98d4b89c7-wdkwx
4m37s       Normal    Pulled              pod/go-sample-web1-98d4b89c7-wdkwx    Container image "azax1234/go-webapp-sample:0.0.1.RELEASE" already present on machine
4m37s       Normal    Created             pod/go-sample-web1-98d4b89c7-wdkwx    Created container go-webapp-sample
4m37s       Normal    Started             pod/go-sample-web1-98d4b89c7-wdkwx    Started container go-webapp-sample
8s          Normal    ScalingReplicaSet   deployment/go-sample-web1             Scaled up replica set go-sample-web1-98d4b89c7 to 3 from 1
7s          Warning   FailedScheduling    pod/go-sample-web1-98d4b89c7-jvjwr    0/1 nodes are available: 1 Insufficient cpu. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod..
7s          Warning   FailedScheduling    pod/go-sample-web1-98d4b89c7-k8rsz    0/1 nodes are available: 1 Insufficient cpu. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod..
7s          Normal    SuccessfulCreate    replicaset/go-sample-web1-98d4b89c7   Created pod: go-sample-web1-98d4b89c7-k8rsz
7s          Normal    SuccessfulCreate    replicaset/go-sample-web1-98d4b89c7   Created pod: go-sample-web1-98d4b89c7-jvjwr
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get events --sort-by=.metadata.creationTimestamp
LAST SEEN   TYPE      REASON              OBJECT                                MESSAGE
4m58s       Normal    Killing             pod/go-sample-web1-98d4b89c7-tcjr5    Stopping container go-webapp-sample
4m54s       Normal    SuccessfulCreate    replicaset/go-sample-web1-98d4b89c7   Created pod: go-sample-web1-98d4b89c7-wdkwx
4m54s       Normal    Scheduled           pod/go-sample-web1-98d4b89c7-wdkwx    Successfully assigned default/go-sample-web1-98d4b89c7-wdkwx to gk3-my-cluster-pool-2-5c363fdd-lbsy
4m53s       Normal    Pulled              pod/go-sample-web1-98d4b89c7-wdkwx    Container image "azax1234/go-webapp-sample:0.0.1.RELEASE" already present on machine
4m53s       Normal    Created             pod/go-sample-web1-98d4b89c7-wdkwx    Created container go-webapp-sample
4m53s       Normal    Started             pod/go-sample-web1-98d4b89c7-wdkwx    Started container go-webapp-sample
24s         Normal    ScalingReplicaSet   deployment/go-sample-web1             Scaled up replica set go-sample-web1-98d4b89c7 to 3 from 1
23s         Warning   FailedScheduling    pod/go-sample-web1-98d4b89c7-k8rsz    0/1 nodes are available: 1 Insufficient cpu. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod..
23s         Warning   FailedScheduling    pod/go-sample-web1-98d4b89c7-jvjwr    0/1 nodes are available: 1 Insufficient cpu. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod..
23s         Normal    SuccessfulCreate    replicaset/go-sample-web1-98d4b89c7   Created pod: go-sample-web1-98d4b89c7-k8rsz
23s         Normal    SuccessfulCreate    replicaset/go-sample-web1-98d4b89c7   Created pod: go-sample-web1-98d4b89c7-jvjwr
13s         Normal    TriggeredScaleUp    pod/go-sample-web1-98d4b89c7-k8rsz    pod triggered scale-up: [{https://www.googleapis.com/compute/v1/projects/sonic-ivy-424616-s5/zones/us-east1-b/instanceGroups/gk3-my-cluster-pool-2-01392188-grp 0->1 (max: 1000)}]
13s         Normal    TriggeredScaleUp    pod/go-sample-web1-98d4b89c7-jvjwr    pod triggered scale-up: [{https://www.googleapis.com/compute/v1/projects/sonic-ivy-424616-s5/zones/us-east1-b/instanceGroups/gk3-my-cluster-pool-2-01392188-grp 0->1 (max: 1000)}]
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get rs
NAME                       DESIRED   CURRENT   READY   AGE
go-sample-web1-98d4b89c7   3         3         1       8d
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get pods 
NAME                             READY   STATUS    RESTARTS   AGE
go-sample-web1-98d4b89c7-jvjwr   0/1     Pending   0          41s
go-sample-web1-98d4b89c7-k8rsz   0/1     Pending   0          41s
go-sample-web1-98d4b89c7-wdkwx   1/1     Running   0          5m12s
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get pods -o wide
NAME                             READY   STATUS    RESTARTS   AGE     IP           NODE                                  NOMINATED NODE   READINESS GATES
go-sample-web1-98d4b89c7-jvjwr   0/1     Pending   0          53s     <none>       <none>                                <none>           <none>
go-sample-web1-98d4b89c7-k8rsz   0/1     Pending   0          53s     <none>       <none>                                <none>           <none>
go-sample-web1-98d4b89c7-wdkwx   1/1     Running   0          5m24s   10.54.0.91   gk3-my-cluster-pool-2-5c363fdd-lbsy   <none>           <none>
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get pods -o wide
NAME                             READY   STATUS              RESTARTS   AGE     IP           NODE                                  NOMINATED NODE   READINESS GATES
go-sample-web1-98d4b89c7-jvjwr   0/1     ContainerCreating   0          110s    <none>       gk3-my-cluster-pool-2-01392188-ctk2   <none>           <none>
go-sample-web1-98d4b89c7-k8rsz   0/1     ContainerCreating   0          110s    <none>       gk3-my-cluster-pool-2-01392188-ctk2   <none>           <none>
go-sample-web1-98d4b89c7-wdkwx   1/1     Running             0          6m21s   10.54.0.91   gk3-my-cluster-pool-2-5c363fdd-lbsy   <none>           <none>
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ kubectl get pods -o wide
NAME                             READY   STATUS              RESTARTS   AGE     IP           NODE                                  NOMINATED NODE   READINESS GATES
go-sample-web1-98d4b89c7-jvjwr   0/1     ContainerCreating   0          2m12s   <none>       gk3-my-cluster-pool-2-01392188-ctk2   <none>           <none>
go-sample-web1-98d4b89c7-k8rsz   0/1     ContainerCreating   0          2m12s   <none>       gk3-my-cluster-pool-2-01392188-ctk2   <none>           <none>
go-sample-web1-98d4b89c7-wdkwx   1/1     Running             0          6m43s   10.54.0.91   gk3-my-cluster-pool-2-5c363fdd-lbsy   <none>           <none>
azax1234@cloudshell:~ (sonic-ivy-424616-s5)$ 