# Kubernetes exercises
These exercises support the introduction to Kubernetes during the jx-workshop

## Preparations
In order to use minikube during the exercises, execute these preparation steps:

  1. Ensure minikube is runnig

  ```bash
  minikube status
  ```
  should produce the following output:
  ```bash
  host: Running
  kubelet: Running
  apiserver: Running
  kubectl: Correctly Configured: pointing to minikube-vm at [your-minikube-vm-ip]
  ```
  if not, start minikube:
  ```bash
  minikube start
  ```
  wait for the command to complete successfully
  
  Find the minikube-vm IP address
  ```bash
  minikube status
  ```
  the last line of the output shows the IP address, remember this.
  
  1. Ensure kubectl is pointing to your minikube cluster

  ```bash
  kubectl get nodes
  ```
  should produce the following output:
  ```bash
  NAME       STATUS    ROLES     AGE       VERSION
  minikube   Ready     master    [age]     [v1.13.4]
  ```
  if not, switch to the correct context:
  ```bash
  kubectl config get-contexts
  kubectl config set-context minikube
  ```
## Basic kubernetes exercises
These exercises provide some insights into the basic usage of kubernetes using the **Imperative** kubernetes style.

### 1 Deploy to Kubernetes
Follow this Kubernetes guide [Create a deployment](https://kubernetes.io/docs/tutorials/hello-minikube/#create-a-deployment)

### 2 Expose the deployed pod outside the cluster
In order to connect to a service from your local machine it needs to be exposed either via a `NodePort` or a `LoadBalancer`,
this can be achieved using the following command:
```bash
kubectl expose deployment hello-node --type=NodePort --port=8080 --target-port=8080
```

Find the port the service is exposed on:
```bash
kubectl describe svc hello-node
```
Take note of the NodePort value

In the browser go to `http://[your-minikube-vm-ip]:[NodePort]`

## Advanced usage exercises
During the following advanced exercises you will be creating an environment with the snack service including dependencies for the service.

As the service depends on MongoDB (with persistent storage), lets start with that.

### 3 Create namespace for the snacks-environment
In order not to pollute the default namespace, a dedicated namespace per application is adviced. 

To create one for our application run:
```bash
kubectl create namespace snacks-environment
```

### 4 Create persistent storage
Kubernetes persistent storage can be mounted into a pod from a **persistent volume claim** which claims an amount storage (xGb, xMb etc)
from a **persistent volume** which is binds to a local path or a cloud storage device.

To create the persistent storage for the db run:
1. Change the path to your `Kube-Volumes/snacks-db` in `~/jx-workshop/Development/jx-ws-kube/mongo-pv.yaml`

2. Create the PV and PVC
  ```bash
  kubectl apply -f ~/jx-workshop/Development/jx-ws-kube/mongo-pv.yaml
  kubectl apply -f ~/jx-workshop/Development/jx-ws-kube/mongo-pv-claim.yaml --namespace=snacks-environment
  ```
3. Check the availability of the persistent storage
  ```bash
  kubectl get pv
  kubectl get pvc -n snacks-environment
  ```
  
### 5 Deploy db
```bash
kubectl apply -f ~/jx-workshop/Development/jx-ws-kube/mongo-deployment.yaml --namespace=snacks-environment
kubectl apply -f ~/jx-workshop/Development/jx-ws-kube/mongo-service.yaml --namespace=snacks-environment
```
Verify namespace content:
```
kubectl -n snacks-environment get all
```
  
### 6 Deploy snacks micro service
```bash
kubectl apply -f ~/jx-workshop/Development/jx-ws-kube/snacks-deployment.yaml --namespace=snacks-environment
kubectl apply -f ~/jx-workshop/Development/jx-ws-kube/snacks-service.yaml --namespace=snacks-environment
```
Verify namespace content:
```
kubectl -n snacks-environment get all
```
  
## END: Clean up
Once your done exploring kubernetes [kubectl commandline reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) clean up your minikube cluster
```bash
kubectl delete service hello-node
kubectl delete deployment hello-node

kubectl delete namespace snacks-environment
kubectl delete pv mongo-pv-volume
```
