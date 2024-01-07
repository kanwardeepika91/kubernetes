## Commands to Install minikube on Mac  
1. brew update
2. brew install hyperkit # a hypervisor software required to run the kubectl
3. brew install minikube 
4. minikube start -vm-driver=hyperkit # to start the kubernetes cluster
5. kubectl version
6. minikube status # which will give you below  output
 ```
 (minikube status
    minikube
    type: Control Plane
    host: Running
    kubelet: Running
    apiserver: Running
    kubeconfig: Configured)
 ```

### Minikube: 
minikube provisions and manages local Kubernetes clusters optimized for development workflows. 
It installs the kubectl and also the docker if not available

### Kubectl: 
kubectl controls the Kubernetes cluster manager

## Important
Remember the Basic flow -> 
1. You create Deployment by giving the deployment name and a Image you want to create a pod for.  
> command: "kubectl create deployment nginx-deploy --image=nginx"
2. If you want to edit an image and its configuration, you have to do that in Deployment only. 
3. If a Deployment is created, it manages Replicaset, pod

### Basic Kubectl commands:

## to get the nodes, deployment, replicaset,pods, services etc
  1.  kubectl get nodes
  2.  kubectl get pod                # initially you wont find any pods running so create a deployment using 4.
  3.  kubectl get services           # initially you will see only the clusterID

## To Create the Deployment 
  4.  kubectl create deployment nginx-deploy --image=nginx
  5.  kubectl get deployment
  6.  kubectl get pod
  7. kubectl get replicaset         # managed by deployment and this will output the desired and current running no. of pods 
  
## To Edit the Deployment
   
  9. kubectl edit deployment nginx-deploy
 `When you pull the image, all the configurations related to the image will be pulled, make some changes to this file such as image from nginx to nginx:1.25.3 or replicas from 1 to 2.
  
  Here, you don't need to restart the pod, When changes are done, it automatically restarts [old one terminates and a new one will be created] and everything else like pod, replicaset everything will be updated. 

  So everything depends on the initial Deployment or configuration files only.`

 ## Debugging: To Check the logs
 10. kubectl get pod (or) kubectl get pod -o wide    # to get more details with ip address of pod
 11. kubectl logs podname                            # will give you around 20 lines of logs
 12. kubectl describe pod podname                    # will give you enough details about the pod in the form of events
 
## To log inside the Pod/get interactive terminal
13. kubectl exec -it podname -- bin/bash

## To delete the Deployment
14. kubectl delete deployment deploymentname

##### The above all were commands execution but if we want to supply multiple commands while creating the Deployment, then becomes difficult and to avoid that better is to use the configuration file for Deployment creation

# Project1: 1.nginx-project related details below

15. touch nginx-deploy.yaml            # add some configuration to this file or use nginx-deployment.yaml file
16. kubectl apply -f nginx-deployment.yaml                           # deployment will be created
17. kubectl get deployment        
18. kubectl get replicaset      # you will see desired and current as 2 if both running
19. kubectl get pod            # you will see 2 pods running 

## you may change configuration like add more replicas and again try below command
20. kubectl apply -f nginx-deployment.yaml
21. kubectl get pod
22. kubectl get deployment nginx-deployment -o yaml  # to get the output of this deployment(desired and actual status of replicas etc or store in a seperate yml file nginx-deployment-result.yaml)
23. kubectl get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml
24. kubectl delete -f nginx-deployment.yaml     # to delete the deployment as you created it using the file

# Kubernetes Configuration File
 Contains 3 parts
 1. Metadata            # of component that you are creating for Kind: Deployment or Service
 2. Specification       # of component with ports/image
 3. Status              # Kubernetes automatically detects the replicas and matches the actual state to desired state

# Connecting components like Deployments, services using labels and selectors
1. metadata will have labels   # Deployment has two metadata , one for Deployment and other for Pod Specification 
2. Specification will have selector and matches labels to create replicas for that Pod, this way Deployment specification would know for which app it has create replicas.
3. Service will look for metadata labels in 'Deployment' and will have specification accordinly and add ports and have a selector to match label with Deployment.
4. Service also has Ports section
for example : ContainerPort in Deployment -pod specification is 8080
              nginx service listens on port 80 (mentioned in Kind: Service)
              TargetPort (mentioned in Kind:Service should match the ContainerPort and should be 8080 , so the targetport in Service will reach the exposed containerPort on 8080)


# Let's now create Service 
0. kubectl apply -f nginx-deployment.yaml  
1. touch nginx-service.yaml    # add configuration and a selector that must match with labels in Kind:Deployment configuration file
2. kubectl apply -f nginx-service.yaml
3. kubectl get service    
4. kubectl get pod -o wide     # check the IP address
5. kubectl describe service nginx-service       # check the Endpoints (IP address) will match with nginx deployment pods
6. kubectl get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml  # here you can see the status of deployment created that helps in debugging any issue as well if the configuration is setup correctly
7. kubectl delete -f nginx-deployment.yaml     # to delete the deployment as you created it using the file
8. kubectl delete -f nginx-service.yaml 


# Project2 : 2.mongoexpress-mongodb-project
1. Mongo Express -> Website 
2. Mongo DB -> Database

### Requirement Details to configure K8's is :
```
User will browse data and request will goto Mongo Express(External Service)->Mongo Express(Pod) -> MongoDB (Internal Service which is URL) -> MongoDB (pod) and here it authenticates with DB credentials

Write a Deployment configuration file for Mongo Express that listens on port 8080
Write a ConfigMap for MongoDB that contains DB URL, Secrets that contains DB User and Password and map them to Deployment configuration file
Note : DB connection is Internal
Mongo Express is exposed to Internet via 8080
```

###  this whole setup would require: 
2 Deployment configuration one for Mongo Express and one for MongoDB
2 Service configuration file one for Mongo Express[External Service] and one for MongoDB [Internal Service]
1 Configmap
1 Secret              # remember Secret should be created before the deployment if we reference it in mongodb deployment

### Lets now write the configuration files

1. touch mongodb-deployment.yaml   # add the configuration
2. touch mongodb-secret.yaml    # add the secrets and ensure  the secrets are encrypted with base64
3. echo -n 'root'| base64     # this will encode the root and gives encrypted value
4. echo -n 'mongodbpassword1234' | base64  # copy these two username and password values into mongodb-secret.yaml file 
5. kubectl apply -f mongodb-secret.yaml 
6. kubectl get secret
7. kubectl apply -f mongodb-deployment.yaml
8. kubectl get all                          # this will also give you whole list of services and deployments
9. kubectl get all | grep mongodb 
10. touch mongodb-configmap.yaml
11. kubectl apply -f mongodb-configmap.yaml     # contains DB-url , takes it from the mongodb-service
12. touch mongoexpress-deployment.yaml
13. kubectl apply -f mongoexpress-deployment.yaml 
14. kubectl get service           # now, you would see the port 8081:30000 but external-ip is not yet assigned but this is only incase of local setup minikube, else in regular kubernetes setup the External Ip is assigned automatically
15. minikube service mongoexpress-service    # this will ouput the External ip and the URL   


# Project3 : Understand about namespace
 It is a unique name if you want to organize your resources, otherwise its default.
 Namespace divides the cluster resources between multiple user, enabling resource quota management

1. kubectl get ns
        NAME              STATUS   AGE
        default           Active   25h
        kube-node-lease   Active   25h
        kube-public       Active   25h 
        kube-system       Active   25h

default            : all the resouces like deployments, pods etc will be created in default namespace
kube-node-lease    : 
kube-public        : resources here doesn't required any authentication to access, are publicly available
kube-system        : System resources are organized in this kube-system

### Create namespace
1. kubectl create ns deepika-kanwar-project
2. kubectl apply -f nginx-deployment.yaml --namespace=deepika-kanwar-project (OR) do the below  
3. add the namespace: deepika-kanwar-project inside the configuration file in 3.namespace folder->nginx-deployment.yaml under metadata section
4. kubectl apply -f nginx-deployment.yaml
5. kubectl get deployment -n deepika-kanwar-project


# Project4 : Understand Ingress



