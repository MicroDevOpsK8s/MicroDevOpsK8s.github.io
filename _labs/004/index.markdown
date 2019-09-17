---
layout: lab
title: Publish your container
---

In this lab you will learn how to manually push your container into a docker registry on Azure. Later on we will 
start using the DevOps pipeline to do so but for now this is manual.

## Preparation

1. Login to Azure.

1. Create a docker registry.

1. Click on the newly created registry and go to `Settings` -> `Access keys` in the left navigation.

1. Note the parameters you will need to access the registry.

## Login and push
   
1. Login to the docker registry:
   <pre>
   docker login -u &lt;azure registry user&gt; &lt;azure registry host&gt;
   </pre>
   You will be asked for the password interactively. `docker` will prevent you from using the `-p` parameter for 
   security reasons.

1. Tag your image appropriately. To be able to push to that registry, your image must be tagged using the following 
pattern: `<registryname>/<image>:<version>`. While you can omit the version, the registry name and image name are mandatory.
   <pre>
   docker tag demoapp &lt;registryname&gt;/demoapp
   </pre>
   
1. You can now double-check with `docker images`. To to finally upload your image to the registy, use:
   <pre>
   docker push &lt;registryname&gt;/demoapp
   </pre>
   The output should be similar to this:
   <pre>
   $ docker push &lt;registryname&gt;.azurecr.io/demoapp
   The push refers to repository [&lt;registryname&gt;.azurecr.io/demoapp]
   d4a4904ce1b5: Pushed 
   ceaf9e1ebef5: Pushed 
   9b9b7f3d56a0: Pushed 
   f1b5933fe4b5: Pushed 
   latest: digest: sha256:fe8ad6c5c11f9d6659f2ea205fcf4183c4001b53e3c910a491209ce3b710453e size: 1159
   </pre>

## References
* [More Docker fun](https://www.katacoda.com/courses/container-runtimes)