## Component status ( master node)

azax1234@cloudshell:~$ kubectl get componentstatus
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE   ERROR
controller-manager   Healthy   ok        
scheduler            Healthy   ok        
etcd-1               Healthy   ok        
etcd-0               Healthy   ok        
azax1234@cloudshell:~$ kubectl get pods --all-namespaces
NAMESPACE         NAME                                                       READY   STATUS      RESTARTS        AGE
default           go-sample-web1-98d4b89c7-52lmd                             1/1     Running     0               4h9m
default           go-sample-web1-98d4b89c7-86sf2                             1/1     Running     0               4h12m
default           go-sample-web1-98d4b89c7-sqccn                             1/1     Running     0               3h13m
gke-gmp-system    collector-8xxpm                                            2/2     Running     1 (4h9m ago)    4h11m
gke-gmp-system    collector-nrkll                                            2/2     Running     2 (6d16h ago)   6d16h
gke-gmp-system    gmp-operator-5476ccbdd5-92zft                              1/1     Running     0               6d16h
gke-gmp-system    rule-evaluator-75d64dd555-6mphf                            2/2     Running     3 (6d16h ago)   6d16h
gke-managed-cim   kube-state-metrics-0                                       2/2     Running     0               6d16h
kube-system       anetd-w8sd9                                                2/2     Running     0               4h11m
kube-system       anetd-zfjzs                                                2/2     Running     0               6d16h
kube-system       antrea-controller-horizontal-autoscaler-6dfb5849b6-bt985   1/1     Running     0               6d16h
kube-system       egress-nat-controller-7d59d5c4bf-p4pqq                     1/1     Running     0               6d16h
kube-system       event-exporter-gke-54d86d77bb-pnp9n                        2/2     Running     0               6d16h
kube-system       filestore-node-l28xw                                       4/4     Running     0               4h11m
kube-system       filestore-node-s7m57                                       4/4     Running     0               6d16h
kube-system       fluentbit-gke-big-9hpqs                                    2/2     Running     0               6d16h
kube-system       fluentbit-gke-big-m55zj                                    2/2     Running     0               4h11m
kube-system       gcsfusecsi-node-kpl49                                      2/2     Running     0               6d16h
kube-system       gcsfusecsi-node-xzq9f                                      2/2     Running     0               4h11m
kube-system       gke-metadata-server-fnqw8                                  1/1     Running     0               6d16h
kube-system       gke-metadata-server-xjzjn                                  1/1     Running     0               4h10m
kube-system       gke-metrics-agent-jh9qw                                    2/2     Running     0               6d16h
kube-system       gke-metrics-agent-vzkpq                                    2/2     Running     0               4h11m
kube-system       image-package-extractor-7jvs2                              1/1     Running     0               4h10m
kube-system       image-package-extractor-8gttm                              1/1     Running     0               6d16h
kube-system       image-package-extractor-cleanup-28625760-mf7fw             0/1     Completed   0          