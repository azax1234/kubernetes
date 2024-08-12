# Declare phony targets
.PHONY: start_minikube install_minikube install_kubectl create_pv create_pvc deploy_registry expose_registry configure_minikube restart_minikube test_registry


MINIKUBE_IP := $(shell minikube ip)
REGISTRY_PORT := $(shell kubectl get svc registry -o jsonpath='{.spec.ports[0].nodePort}')

# Define the default target
#all: build


install_minikube:
	@echo "Installing minikube..."
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && \
    sudo install minikube-linux-amd64 /usr/local/bin/minikube && \
    rm minikube-linux-amd64


install_kubectl:
	@echo "installing kubectl..."
	curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
	kubectl get all

start_minikube: install_minikube install_kubectl
	@echo "Starting minikube..."
	minikube start

create_pv:
	kubectl apply -f - << 'EOF'
	apiVersion: v1
	kind: PersistentVolume
	metadata:
	  name: registry-pv
	spec:
	  capacity:
	    storage: 5Gi
	  accessModes:
	    - ReadWriteOnce
	  hostPath:
	    path: "/mnt/data"
	 'EOF'

create_pvc:
	kubectl apply -f - << 'EOF'
	apiVersion: v1
	kind: PersistentVolumeClaim
	metadata:
	  name: registry-pvc
	spec:
	  accessModes:
	    - ReadWriteOnce
	  resources:
	    requests:
	      storage: 5Gi
	 'EOF'

deploy_registry:
	kubectl apply -f - << 'EOF'
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: registry
	spec:
	  replicas: 1
	  selector:
	    matchLabels:
	      app: registry
	  template:
	    metadata:
	      labels:
	        app: registry
	    spec:
	      containers:
	      - name: registry
	        image: registry:2
	        ports:
	        - containerPort: 5000
	        volumeMounts:
	        - name: storage
	          mountPath: /var/lib/registry
	      volumes:
	      - name: storage
	        persistentVolumeClaim:
	          claimName: registry-pvc
	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: registry
	spec:
	  selector:
	    app: registry
	  ports:
	  - port: 5000
	    targetPort: 5000
	 'EOF'

expose_registry:
	kubectl expose deployment registry --type=NodePort --port=5000

configure_minikube:
	minikube ssh "sudo sed -i 's|ExecStart=/usr/bin/dockerd|ExecStart=/usr/bin/dockerd --insecure-registry=$(MINIKUBE_IP):$(REGISTRY_PORT)|' /lib/systemd/system/docker.service"
	minikube ssh "sudo systemctl daemon-reload && sudo systemctl restart docker"

restart_minikube:
	minikube stop
	minikube start

test_registry:
	docker pull nginx:latest
	docker tag nginx:latest $(MINIKUBE_IP):$(REGISTRY_PORT)/my-nginx:latest
	docker push $(MINIKUBE_IP):$(REGISTRY_PORT)/my-nginx:latest
	docker rmi $(MINIKUBE_IP):$(REGISTRY_PORT)/my-nginx:latest
	docker pull $(MINIKUBE_IP):$(REGISTRY_PORT)/my-nginx:latest

setup_registry: start_minikube create_pv create_pvc deploy_registry expose_registry configure_minikube restart_minikube test_registry