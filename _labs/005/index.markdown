---
layout: lab
title: Azure DevOps Pipelines
---

In this lab we will find our way through Azure DevOps. The main steps will be:
* adding the code to a new Azure Repo
* creating a pipeline to build and upload the docker image to the registry

## Preparation

1. Login to [Azure DevOps](https://dev.azure.com/) and switch to your organization.


## Git magic
   
1. Create a new, empty repository in Azure DevOps, possibly named `demoapp`.
1. Make sure you created an ssh key with `ssh-keygen` for your local machine and uploaded the public key (`id_rsa.pub`)
to your Azure DevOps profile, so that you can use git and ssh without being nagged with passwords.
to your Azure DevOps profile, so that you can use git and ssh without being nagged with passwords.

1. As your local source directory isn't connected to any git repository, we'll need to do this now. First, initialize
your demoapp directory:
    <pre>
    $ cd &lt;sourcedir&gt;
    $ git init
    $ git add .
    $ git commit -m "Adding the demo app to git"
    </pre>
    
1. In Azure DevOps, if you click on your newly created, empty repo, follow the instructions for 
    **push an existing repository from command line** and click on the `SSH` tab and follow the instructions:
    <pre>
    git remote add origin git@ssh.dev.azure.com:v3/&lt;yourorg&gt;/&lt;yourproject&gt;/demoapp
    git push -u origin --all
    </pre>
    
    Your code should now be in an Azure git repository.

## Pipelines
The idea of a pipeline is to have some automation kicked off on each and every `git push` or creation and merge of a 
git pull request.
While many projects use [TravisCI](https://travis-ci.org/) or a separate [Jenkins server](https://jenkins.io/) to do 
that part of automation, Azure uses its own pipeline.
This requires that in your repository there is a specific file named `azure-pipelines.yaml`, which we will now create,
step-by-step.
Limited local testing can be achieved by runnig a local [pipeline agent](https://github.com/microsoft/azure-pipelines-agent).

1. First, we need to create a connection to the Azure registry with a service connection. To do so, go to Azure DevOps
and click on _Project Settings_ at the botton left of your browser window, then click on _Service connections_ under **Pipelines**.

1. Click on _New Service connection_ and select **Docker Registry**. In the pop-up window, select _Azure Container Registry_
and provide the relevant data.

1. On your local machine, go to your code's root directory and create a new file named `azure-pipelines.yml` with the following
   content:
   <pre>
    trigger:
      - master
    
    pool:
      vmImage: 'ubuntu-latest'
    
    variables:
      imageName: demoapp:$(build.buildId)
    
    steps:
      - task: Gradle@2
        inputs:
          jdkVersionOption: '1.8'
          tasks: 'build'
    </pre> 
    
1. On the command line, check in the new file:
   <pre>
   $ git add azure-pipelines.yml
   $ git commit -m "added pipeline"
   $ git push
   </pre>
   
1. In the Azure DevOps portal, click on _Pipelines_ and hit the **New pipeline** button on the top right.
1. On the following page, select _Azure Repos Git_, then select your git repo with the newly created `azure-pipelines.yml`
   and select **Existing Azure Pipelines YAML file**.
1. Provide the branch and path to the pipelines file: `/azure-pipelines.yml` and hit **Continue**
1. Your `azure-pipelines.yml` is shown and to manually run it, hit the **Run** button on the top right of the page.
1. Each pipeline has several **Jobs**, one for each build taking place. Click on the latest job and inspect the steps.

##Extend the pipeline

1. Now that the build is working, we want to create a docker image and push it to the docker registry. This can be done by adding
   the following code to the variables section and to the end of the `azure-pipelines.yml`:
   <pre>
     :
     variables:
       dockerRegistry: &lt;your registry name&gt;
       :
       - task: Docker@2
         displayName: Login to ACR
         inputs:
           command: login
           containerRegistry: &lt;your registry connnection&gt;
       - task: Docker@2
         displayName: Build and Push
         inputs:
           command: buildAndPush
           containerRegistry: &lt;your registry connnection&gt;
           repository: $(dockerRegistry).azurecr.io   
   </pre>
   Now, here's the challenge of the day: YAML files rely on indention, so be perfectly sure everything is perfectly
   indented otherwise your pipeline either won't work or will do funny things you didn't exactly ask for.
   
1. Check in the modified `azure-pipelines.yml`:
   <pre>
   $ git add azure-pipelines.yml
   $ git commit -m "added docker build"
   $ git push
   </pre>
   
   Once you pushed your changes, you can see the pipeline being run with a new job. Inspect the results to see what
   happened in either good or bad case :-)
   
   


## References
* [More Docker fun](https://www.katacoda.com/courses/container-runtimes)