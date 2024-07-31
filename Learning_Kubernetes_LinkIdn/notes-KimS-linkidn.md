## Notess from the LinkIdn course by Kim Schlesinger

### yaml files

Yaml is a data serialization language hta makes your data portable. Other notable data serialization languages are json, xml. Yaml is used commonly to create Kubernetes manifests.

--- in yaml stats the begining of a document. You can combine mutiple files ina  single file using --- in the begining of each document.

comments start with a #

You can validate syntax by using sites like [text](https://yamlchecker.com)


## Validating if app is running fine within a pod

install ***busybox*** in default namespace and kubectl exec into it and use wget with ip:port of the pod

kubectl exec -it busybox-54f785c7d7-c2k6w  -- /bin/sh

 wget 10.244.0.6:3000

