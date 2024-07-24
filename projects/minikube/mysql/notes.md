# secrets

## Error: secret "mysql-pass" not found in k8 even though specified in the same  directory in kustomization.yaml

Here are some steps to troubleshoot and resolve this issue:

Verify Secret Creation:
Ensure that the secret mysql-pass is created in your Kubernetes cluster. You can check for existing secrets with the following command:

```sh
kubectl get secrets
```
Check Secret Definition in kustomization.yaml:
Verify that the secret is correctly defined in your kustomization.yaml file. It should look something like this:

yaml

secrets:
- name: mysql-pass
  literals:
  - password=yourpassword
Alternatively, if you are using a file for the secret, it should look like this:

yaml

secrets:
- name: mysql-pass
  files:
  - password=path/to/password/file
Apply Kustomization:
Make sure you apply the kustomization directory with the correct path. Run the following command from the directory where your kustomization.yaml is located:

sh

kubectl apply -k .
Namespace Check:
Ensure that the secret is created in the correct namespace. If you are working in a namespace other than default, make sure to include the namespace in your commands:

sh

kubectl apply -k . -n your-namespace
Inspect Existing Secrets:
Check the existing secrets in the namespace where you expect the secret to be:

sh

kubectl get secrets -n your-namespace
Reapply the Secret:
If the secret is missing, you can reapply the kustomization to ensure the secret gets created:

sh

kubectl delete secret mysql-pass -n your-namespace
kubectl apply -k . -n your-namespace

# Using the mysql DB after it is  deployed 

## directly via the pod 
```bash
kubectl exec -it  wordpress-mysql-76fccd887b-wrkdj  -- bash
```
## port -forward

sh

kubectl port-forward svc/mysql 3306:3306
Then, you can connect to MySQL locally as if it were running on your local machine:

sh

mysql -h 127.0.0.1 -P 3306 -u <username> -p<password>


## Create a LoadBalancer or NodePort Service

You can expose the MySQL service externally by changing the service type to LoadBalancer or NodePort. However, this is generally not recommended for production databases due to security concerns.

Example of a NodePort Service:

yaml

apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  ports:
    - port: 3306
      nodePort: 32000
  selector:
    app: mysql
After applying this configuration, you can access MySQL using the IP address of any node in the cluster and the assigned node port (e.g., 32000):

sh

mysql -h <node-ip> -P 32000 -u <username> -p<password>
Example of a LoadBalancer Service:

yaml

apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: LoadBalancer
  ports:
    - port: 3306
  selector:
    app: mysql
The cloud provider will assign an external IP to the LoadBalancer, and you can access MySQL using that external IP and port 3306.

