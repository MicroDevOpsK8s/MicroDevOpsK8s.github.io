---
layout: lab
title: Azure DevOps Deployment
---

In this lab we will create a tiny, single node kubernetes cluster, and deploy our code to it. In the following, 
we'll use K8s (Kubernetes) or AKS (Azure Kubernetes Service) to shorten things...

## Preparation

1. Login to [Azure Portal](https://portal.azure.com/) and switch to your organization.


## Create a new Kubernetes cluster
   
1. Click on **Create a resource** on the upper left corner and search for `kubernetes` in the search field, then select
   `Kubernetes Service` from the list.
1. Click **Create**
1. On the following pages, fill in all mandatory fields, be sure to select the cheapest possible **Node size**. 
We don't need much computing power for our demo app so the smallest possible configuration will do. As of writing 
this, this is named `B2s`. 
<br>Also be sure to only select 1 node (default is 3)!!
1. Click **Review + create** and once it passes validation, hit the **Create** button and grab a coffee....


## Creating the deployment YAML 

###Background
Unlike a VM where you can `ssh` into and run bash scripts and all this, kubernetes is steered via an API server (the Kubernetes
master node). Almost the only tool you will ever need is `kubectl` to manage deployments and configuration. And - almost
all of these actions are driven by sending a specifically tailored `.yml` (or `.yaml`) file with the help of `kubectl`.

1. As we first want to test our deployment yaml, we will install a very basic k8s _cluster_ (Don't call it _cluster_ anyways...)
named [minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/). 
<br>So, click the link above and follow the instructions for your operating system.
1. Fire it up by running `minikube start` and wait for it to com up.
1. Check that everything is smooth by running `minikube kubectl get ns`. The output should be similar to this:
   <pre>
    $ minikube kubectl get ns
    💾  Downloading kubectl v1.15.2
    NAME              STATUS   AGE
    default           Active   95s
    kube-node-lease   Active   98s
    kube-public       Active   98s
    kube-system       Active   98s
   </pre>

1. Create a subdirectory named `manifests` and create preliminary deployment.yaml in that directory with the following content:
   <pre>
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: demoapp
     labels:
       app: demoapp
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: demoapp
     template:
       metadata:
         labels:
           app: demoapp
       spec:
         containers:
           - name: demoapp
             imagePullPolicy: IfNotPresent
             image: nemoregistrysandbox.azurecr.io/demoapp
             ports:
               - containerPort: 8080
   </pre>
   **NOTE**: Minikube is running in a virtual machine (at least on MacOS and Windows) and therefore it's isolated
   from the host operating system(yes, your laptop) for security reasons. Our deployment.yml file references an image
   named `demoapp`, which indicates, that it should look in a local registry. The local registry for minikube unfortunately
   is the one within the VM. So now we will turn things around and expose the VM's docker registry to our laptop.
   To do so, run 
   <pre>
   $ eval $(minikube docker-env)
   </pre>
   to set your environment to point to minikube's docker registry and re-run the build:
   <pre>
   $ docker build -t demoapp --build-arg JAR_FILE=build/libs/demo-0.0.1-SNAPSHOT.jar .
   </pre>
    
1. Minikube is running in a virtual machine, so it is isolated from the host (your laptop) and therefore you
   cannot access the ports directly. The easiest way to expose the port is by using the `port-forward`.
   <pre>
   $ kubectl get pods
   NAME                       READY   STATUS    RESTARTS   AGE
   demoapp-84b67c64c8-24lvj   1/1     Running   0          2d
   </pre>
   Then note the pod name and run:
   <pre>
   $kubectl port-forward demoapp-84b67c64c8-24lvj 8080
   </pre>
   Now you'll be able to `curl` to your pod:
   <pre>
   $ curl -X GET http://localhost:8080/hello 
   </pre>
   
1. Challenging Exercise - somewhat
   Try to modify the deployment.yaml and scale the service to 2 instances. You should then also try to create a 
   kubernetes service. A service in k8s terms is a loadbalancer that distributes the load to your different pods.

##Extend the pipeline

Now that the k8s deployment is working, the k8s cluster is up'n running, we want to deploy our code to AKS. 
Doing so requires a few steps:
 * Creating the cluster
 * Allow AKS to connect to the ACR (Azure Container Registry)
 * Create image pull secrets, so that AKS has the credentials to connect to ACR
 * Creating a Release pipeline

<br>NOTE: All these steps are in the Microsoft documentation 
 
1. Since we already created the cluster, we can continue with the second step which is outlined [here](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-aks#grant-aks-access-to-acr).
   Create a new file (shell script) with the following content:
   <pre>
   #!/bin/bash
   
   AKS_RESOURCE_GROUP=myAKSResourceGroup
   AKS_CLUSTER_NAME=myAKSCluster
   ACR_RESOURCE_GROUP=myACRResourceGroup
   ACR_NAME=myACRRegistry
   
   # Get the id of the service principal configured for AKS
   CLIENT_ID=$(az aks show --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv)
   
   # Get the ACR registry resource id
   ACR_ID=$(az acr show --name $ACR_NAME --resource-group $ACR_RESOURCE_GROUP --query "id" --output tsv)
   
   # Create role assignment
   az role assignment create --assignee $CLIENT_ID --role acrpull --scope $ACR_ID
   </pre>
   Save the file, make it executable with:
   <pre>
   $ chmod +x &lt;scriptname&gt;
   </pre>
   Afterwards, login to AKS and set the subscription:
   <pre>
   $ az login -u &lt;your user name&gt;
   $ az account set -s &lt;your subscription&gt;
   </pre>
   Note that you will be challenged for your password. upon login.
   <br>**_Then execute your grant script from above_**
   
1. Create a new service connection of type _Azure Resource Manager_ in Azure DevOps and add the following content
   to your `azure-pipelines.yml`:
   <pre>
   variables:
       azureSubscriptionEndpoint: &lt;your new service connection&gt;
       azureContainerRegistry: &lt;yourregistry&gt;.azurecr.io
       azureResourceGroup: &lt;your resource group&gt;
       kubernetesCluster: &lt;your clustername&gt;
   steps:
     - task: Kubernetes@1
       displayName: 'kubectl login'
       inputs:
         connectionType: 'Azure Resource Manager'
         azureSubscriptionEndpoint: $(azureSubscriptionEndpoint)
         azureResourceGroup: $(azureResourceGroup)
         kubernetesCluster: $(kubernetesCluster)
         command: login
    
     - task: Kubernetes@1
       displayName: kubectl apply
       inputs:
         connectionType: 'Azure Resource Manager'
         azureSubscriptionEndpoint: $(azureSubscriptionEndpoint)
         azureResourceGroup: $(azureResourceGroup)
         kubernetesCluster: $(kubernetesCluster)
         command: apply
         arguments: -f manifests/deployment.yml
   </pre>
   
1. You will now discover that your deployment works fine, but looking at the cluster with:
   <pre>
   $ kubectl get pods
   </pre>
   you will discover that your pod has a state of `ImagePullBackOff` or `ErrImagePull`. If you dig deeper with:
   <pre>
   $ kubectl describe pod &lt;pod name&gt; 
   </pre>
   the event log will tell you that it failed to pull the docker image, with an authentication problem.
   To fix this, you need to extend the `deployment.yml` from:
   <pre>
   :
        containers:
          - name: demoapp
            imagePullPolicy: IfNotPresent
            image: nemoregistrysandbox.azurecr.io/demoapp
            ports:
              - containerPort: 8080
   </pre>
   with the pullsecrets:
   <pre>
   :
        containers:
          - name: demoapp
            imagePullPolicy: IfNotPresent
            image: nemoregistrysandbox.azurecr.io/demoapp
            ports:
              - containerPort: 8080
        imagePullSecrets:
          - name: acr-auth
   </pre>
   Now, that you reference the `act-auth` image pull secrets, you need to define them within the cluster. This can be achieved
   by different methods. First, by using `kubectl` on the command line. However, this would be a manual step that we 
   need to avoid (at all cost, to be honest). 
   <br>So, instead we're creating a YAML file that we can deploy in an automated way. Therefore we need to create a
   new yaml file that will let us deploy the secrets. 
1. Creating this yaml file requires that you first login to that docker registry with `docker login`, which then creates
   a `.docker/config.json` file in your HOME directory. Run the follwing command and copy the output:  
   <pre>
   $ cat ~/.docker/config.json | base64
   </pre>
   <br>Create a new file named `manifests/pullsecrets.yml` with the following content:
   <pre>
   apiVersion: v1
   kind: Secret
   metadata:
     name: acr-pullsecrets
     namespace: default
   data:
     .dockerconfigjson: &lt;your base64 encoded ~/&lt;user&gt;/.docker/config.json file&gt;
   type: kubernetes.io/dockerconfigjson
  
   </pre>
1. Check-in your changes to git and [push-it](https://giphy.com/gifs/people-help-5YS2veXdeDhsI).
1. If everything went ok, let's do some `kubectl` magic:
   <pre>
   $ kubectl get ns
   $ kubectl get pods --all-namespaces
   $ kubectl describe pod &lt;pod name&gt;
   $ kubectl port-forward &lt;pod name&gt; 8080
   </pre>
1. Once you port-forwarded the pod, you should be able to access it via `curl`:
   <pre>
   $ curl -X GET http://localhost:8080/hello
   </pre>


## References
* [AKS Introduction](https://docs.microsoft.com/de-de/azure/aks/)