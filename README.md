Commands to Install minikube on Mac  
brew update
brew install hyperkit # a hypervisor ssoftware required to run the kubectl
brew install minikube 
minikube start  # to start the kubernetes cluster
kubectl version
minikube status # which will give you below output
 (minikube status
    minikube
    type: Control Plane
    host: Running
    kubelet: Running
    apiserver: Running
    kubeconfig: Configured)

### Minikube: 
minikube provisions and manages local Kubernetes clusters optimized for development workflows. 
It installs the kubectl and also the docker if not available

### Kubectl: 
kubectl controls the Kubernetes cluster manager

# Important
Remember the Basic flow -> 
1. You create Deployment by giving the deployment name and a Image you want to create a pod for.  
> command: "kubectl create deployment nginx-deploy --image=nginx"
2. If you want to edit an image and its configuration, you have to do that in Deployment only. 
3. If a Deployment is created, it manages Replicaset, pod

### Basic Kubectl commands:

## to get the nodes, deployment, replicaset,pods, services etc
  1.  kubectl get nodes
  2.  kubectl get pod            # initially you wont find any pods running so create a deployment using 4.
  3.  kubectl get services       # initially you will see only the clusterID

## To Create the Deployment 
  4.  kubectl create deployment nginx-deploy --image=nginx
  5.  kubectl get deployment
  6.  kubectl get pod
  7. kubectl get replicaset        # managed by deployment and this will output the desired and current running no. of pods 
  
## To Edit the Deployment
  When you pull the image, all the configurations related to the image will be pulled
  9. kubectl edit deployment nginx-deploy
  make some changes to this file such as image from nginx to nginx:1.25.3 or replicas from 1 to 2.
  Here, you don't need to restart the pod, When changes are done, it automatically restarts [old one terminates and a new one will be created] and everything else like pod, replicaset everything will be updated. So everything depends on the initial Deployment or configuration files only.

 ## Debugging: To Check the logs
 10. kubectl get pod
 11. kubectl logs podname    # will give you around 20 lines of logs
 12. kubectl describe pod podname   # will give you enough details about the pod in the form of events
 
# To log inside the Pod/get interactive terminal
13. kubectl exec -it podname -- bin/bash

# To delete the Deployment
14. kubectl delete deployment deploymentname

# The above all were commands but if we want to supply multiple commands while creating the Deployment, then becomes difficult and to avoid that better is to use the configuration file for Deployment creation

15. touch nginx-deploy.yaml
## add some configuration to this file
16. kubectl apply -f nginx-deployment.yaml                           # deployment will be created
17. kubectl get deployment
18. kubectl get replicaset      # you will see desired and current as 2 if both running
19. kubectl get pod            # you will see 2 pods running 

## you may change configuration like add more replicas and again try below command
20. kubectl apply -f nginx-deployment.yaml
21. kubectl get pod 
