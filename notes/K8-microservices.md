
### Deploying services
-----------------------
ubuntu@ip-172-31-15-80:~/devops/projects$ cd microservices/
ubuntu@ip-172-31-15-80:~/devops/projects/microservices$ ls
01-currency-exchange-microservice-basic  02-currency-conversion-microservice-basic  docker-compose.yml  ingress.yaml  readme.md
ubuntu@ip-172-31-15-80:~/devops/projects/microservices$ cd 01-currency-exchange-microservice-basic/
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/01-currency-exchange-microservice-basic$ ls
Dockerfile  configuration  pipeline-backups  pom.xml  readme.md  src
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/01-currency-exchange-microservice-basic$ find . -name deployment*
./configuration/kubernetes/deployment.yaml
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/01-currency-exchange-microservice-basic$ cd ./configuration/kubernetes/
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/01-currency-exchange-microservice-basic/configuration/kubernetes$ ls
deployment.yaml
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/01-currency-exchange-microservice-basic/configuration/kubernetes$ kubectl apply -f deployment.yaml 
deployment.apps/currency-exchange created
service/currency-exchange created
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/01-currency-exchange-microservice-basic/configuration/kubernetes$ cd ../..
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/01-currency-exchange-microservice-basic$ cd ../02-currency-conversion-microservice-basic/
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/02-currency-conversion-microservice-basic$ ls
00-configmap-currency-conversion.yaml  Dockerfile  deployment.yaml  pom.xml  readme.md  src
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/02-currency-conversion-microservice-basic$ kubectl apply -f deployment.yaml 
deployment.apps/currency-conversion created


ubuntu@ip-172-31-15-80:~/devops/projects/microservices/02-currency-conversion-microservice-basic$ kubectl get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/currency-conversion-846d8754fd-c9wwd   1/1     Running   0          68s
pod/currency-conversion-846d8754fd-cjncs   0/1     Pending   0          68s
pod/currency-exchange-59f85d9b8b-zrgkz     0/1     Running   0          103s
NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)          AGE
service/currency-conversion   LoadBalancer   34.118.235.99   34.23.188.7      8100:32614/TCP   68s
service/currency-exchange     LoadBalancer   34.118.232.86   104.196.36.186   8000:30878/TCP   103s
service/kubernetes            ClusterIP      34.118.224.1    <none>           443/TCP          12d
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/currency-conversion   1/2     2            1           68s
deployment.apps/currency-exchange     0/1     1            0           103s
NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/currency-conversion-846d8754fd   2         2         1       68s
replicaset.apps/currency-exchange-59f85d9b8b     1         1         0       103s
# Now hit the urls

http://104.196.36.186:8000/currency-exchange/from/EUR/to/INR

# Response
{
  "id": 10002,
  "from": "EUR",
  "to": "INR",
  "conversionMultiple": 75,
  "exchangeEnvironmentInfo": "currency-exchange-59f85d9b8b-zrgkz v1 zrgkzupdated"
}


http://34.23.188.7:8100/currency-conversion/from/EUR/to/INR/quantity/10

# Response
{
  "id": 10002,
  "from": "EUR",
  "to": "INR",
  "conversionMultiple": 75,
  "quantity": 10,
  "totalCalculatedAmount": 750,
  "exchangeEnvironmentInfo": "currency-exchange-59f85d9b8b-zrgkz v1 zrgkzupdated",
  "conversionEnvironmentInfo": "currency-conversion-846d8754fd-gjc8j v1 gjc8j"
}


### How does the Curreny Converter microservice able to talk to the Exchange microservice??
------------------------------------------------------------------------------------------
## In the converter config, a variable was specied to look up for the exchange service host


# When we start the exchange microservice, Kubernetes creates a env variable by for the host by adding _SERVICE_HOST to the service name in CAPS and pass it to all other services.
## So the converter service already know which host the exchange service is running

# We can see from below service logs as well
# All service running
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/02-currency-conversion-microservice-basic$ kubectl get service
NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)          AGE
currency-conversion   LoadBalancer   34.118.235.99   34.23.188.7      8100:32614/TCP   32m
currency-exchange     LoadBalancer   34.118.232.86   104.196.36.186   8000:30878/TCP   33m
kubernetes            ClusterIP      34.118.224.1    <none>           443/TCP          12d
## logs for exchange service
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/02-currency-conversion-microservice-basic$ kubectl logs service/currency-exchange | grep HOST
2024-06-09 00:30:14.495  INFO 1 --- [           main] i.m.c.u.e.EnvironmentConfigurationLogger : KUBERNETES_SERVICE_HOST - 34.118.224.1
2024-06-09 00:30:14.590  INFO 1 --- [           main] i.m.c.u.e.EnvironmentConfigurationLogger : CURRENCY_CONVERSION_SERVICE_HOST - 34.118.235.99
2024-06-09 00:30:14.592  INFO 1 --- [           main] i.m.c.u.e.EnvironmentConfigurationLogger : CURRENCY_EXCHANGE_SERVICE_HOST - 34.118.232.86
2024-06-09 00:30:14.598  INFO 1 --- [           main] i.m.c.u.e.EnvironmentConfigurationLogger : HOSTNAME - currency-exchange-59f85d9b8b-zrgkz
## Logs for converter service

ubuntu@ip-172-31-15-80:~/devops/projects/microservices/02-currency-conversion-microservice-basic$ kubectl logs service/currency-conversion | grep HOST
Found 2 pods, using pod/currency-conversion-846d8754fd-cjncs
2024-06-09 00:31:43.714  INFO [currency-conversion,,,] 1 --- [           main] i.m.c.u.e.EnvironmentConfigurationLogger : KUBERNETES_SERVICE_HOST - 34.118.224.1
2024-06-09 00:31:43.716  INFO [currency-conversion,,,] 1 --- [           main] i.m.c.u.e.EnvironmentConfigurationLogger : CURRENCY_CONVERSION_SERVICE_HOST - 34.118.235.99
2024-06-09 00:31:43.717  INFO [currency-conversion,,,] 1 --- [           main] i.m.c.u.e.EnvironmentConfigurationLogger : CURRENCY_EXCHANGE_SERVICE_HOST - 34.118.232.86
2024-06-09 00:31:43.723  INFO [currency-conversion,,,] 1 --- [           main] i.m.c.u.e.EnvironmentConfigurationLogger : HOSTNAME - currency-conversion-846d8754fd-cjncs


### Service Discovery and Load balancing
----------------------------------------
If you provide the host variables like this, what if the echnage service is started after the conversion service. K8 wont pass the env variable in this case explicitely
The better way to do is to update the deployment file of the conversion service with a env variable http://currency-exchange

          - name: CURRENCY_EXCHANGE_SERVICE_HOST
             value: http://currency-exchange
        
            valueFrom: 
             configMapKeyRef:
               key: CURRENCY_EXCHANGE_SERVICE_HOST
               name: currency-conversion-config-map

## When we do a service name passing like this as env var, K8 will automatically do the Service discovery and also Load balancing for us.
## So , if the exchange service now has 2 pods, instead of 1, converter will load balance between both pods.


### Centralised Config management
-----------------------------------
 WE can create a config map and keep all configs centralised
 00-configmap-currency-conversion.yaml

	 `apiVersion: v1
	data:
	  CURRENCY_EXCHANGE_SERVICE_HOST: http://currency-exchange
	kind: ConfigMap
	metadata:
	  name: currency-conversion-config-map
	  namespace: default`

## We can deply this yaml as  a config map
ubuntu@ip-172-31-15-80:~/devops/projects/microservices/02-currency-conversion-microservice-basic$ kubectl apply -f 00-configmap-currency-conversion.yaml 
configmap/currency-conversion-config-map created
## See the configmap craeted. All services with the env value refernce willl read from here when specified. 
$ kubectl get configmaps
NAME                             DATA   AGE
currency-conversion-config-map   1      85s
kube-root-ca.crt                 1      12d


 ## in the deployment file of the service, we can ask it to pick it up from the config map

  - name: CURRENCY_EXCHANGE_SERVICE_HOST
        
          `  valueFrom: 
             configMapKeyRef:
               key: CURRENCY_EXCHANGE_SERVICE_HOST
               name: currency-conversion-config-map`


## INGRESS
------------

Load Balancers are really expensive as these have to be higly available. Its is very expensive to have 1000 LBs for 1000 services.
Instead we can create a Ingress and route to appropiate services.

# Change the service type to NodePort in all deployment.yaml filrs

and apply the ingress.yaml

`ubuntu@ip-172-31-15-80:~/devops/projects/microservices$ kubectl apply -f ingress.yaml 
ingress.networking.k8s.io/gateway-ingress created
ubuntu@ip-172-31-15-80:~/devops/projects/microservices$ kubectl get ingress
NAME              CLASS    HOSTS   ADDRESS   PORTS   AGE
gateway-ingress   <none>   *                 80      13s
`
# kubectl get ingress
NAME              CLASS    HOSTS   ADDRESS         PORTS   AGE
gateway-ingress   <none>   *       34.111.47.228   80      10m

Now we can access both exchange and conversion service by using same ingress url

http://34.111.47.228/currency-exchange/from/EUR/to/INR
http://34.111.47.228/currency-conversion/from/EUR/to/INR/quantity/100

# details of ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gateway-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: / #forward the path as it is
spec:
  rules:
  - http:
      paths:
      - path: /currency-exchange/
        pathType: Prefix
        backend:
          service:
           name: currency-exchange #forward all urls with above path to this service
           port:
              number: 8000     
      - path:  /currency-conversion/
        pathType: Prefix
        backend:
          service: 
           name: currency-conversion
           port:
              number: 8100 

