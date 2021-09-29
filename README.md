# Custom Kubernetes autoscaling algorithm
### ENVIRONMENT SETUP
#### Description
This project is based on Kubernetes, Minikube version v1.19.0, kubectl version v1.21.0, Docker version 20.10.2 running on a Linux Mint 20 Virtual Machine.
#### Setup 
The first software needed to be installed is the host for containers: Docker.
Docker’s containers can be orchestrated and deployed using Kubernetes. In order to set up a single-node Kubernetes cluster on a local machine, I can use Minikube.
Minikube can be installed using:
```sh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
Minikube requires a virtual machine Hypervisor, in this case, the Hypervisor is Docker itself.
```sh
minikube config set driver docker
```
The cluster can be accessed using the command-line tool kubectl.
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
Now it is possible to start Minikube
```sh
minikube start
```
### Horizontal Pod Autoscaler example
Walkthrough of Horizontal Pod autoscaler at https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
This example sends an infinite loop of queries to the php-apache service.
```sh
kubectl apply -f https://k8s.io/examples/application/php-apache.yaml
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
In order to see  how the autoscaler reacts to load I need to enable metrics-server, and get hpa information.
```sh
minikube addons enable metrics-server
kubectl get hpa
```
### Autoscaling on multiple metrics and custom metrics
Start minikube without metrics server ( no need to scale on cpu or memory)
```sh
minikube start &&  minikube dashboard && minikube addons disable metrics-server
```
Install the prometheus stack
```sh
helm install [NAME] prometheus-community/kube-prometheus-stack
```
Port-forward Prometheus and Grafana dashboard:
```sh
kubectl port-forward svc/mon-kube-prometheus-stack-prometheus 9090
kubectl port-forward service/mon-grafana 3000:80
```
Install Prometheus-adapter using custom configuration:
```sh
helm install prometheus-adapter  prometheus-community/prometheus-adapter -f “CUSTOM_PROMETHEUS_ADAPTER_CONFIGURATION.yaml”
```
use “CUSTOM_PROMETHEUS_ADAPTER_CONFIGURATION.yaml”
Apply deploy, service, service monitor and hpa with custom metrics for php-apache.
```sh
kubectl create -f "application-deploy.yaml"
```
use application-deploy.

# END
