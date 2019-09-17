---
layout: lab
title: Package the code
---

In this lab you will learn how to package your code into a docker container and test it.

## Package your code

1. Create a new file named `Dockerfile` in your projects root directory at `demo/`.

1. Add the following code to your newly created file:

   <pre>
   FROM openjdk:8-jdk-alpine
   VOLUME /tmp
   ARG JAR_FILE
   COPY ${JAR_FILE} app.jar
   ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
   </pre>

2. The `FROM` statement references the base container image you want to use. As we want to run Kotlin code which is 
using a JVM, we picked `openjdk:8-jdk-alpine` but we could actually use any image that has a JVM installed. We could even
start with a vanilla ubuntu image and install the JVM while building the image through respective directives in the 
`Dockerfile`.

2. Now, let's build the docker image. Run
   <pre>
   docker build -t demoapp --build-arg JAR_FILE=build/libs/demo-0.0.1-SNAPSHOT.jar .
   </pre>
   
   where `-t demoapp` is the tag you are providing. You can use this tag to identify your container image and if you are 
   publishing your image to [Docker Hub](https://hub.docker.com) you need to have a specific naming which is preceded with 
   your account name or your registry name, e.g. `<user>/demoapp` or `<registry>/demoapp`.
   
2. The output should be similar to this:
   <pre>
    $ docker build --build-arg JAR_FILE=build/libs/demo-0.0.1-SNAPSHOT.jar .
    Sending build context to Docker daemon  21.91MB
    Step 1/5 : FROM openjdk:8-jdk-alpine
    8-jdk-alpine: Pulling from library/openjdk
    e7c96db7181b: Pull complete 
    f910a506b6cb: Pull complete 
    c2274a1a0e27: Pull complete 
    Digest: sha256:94792824df2df33402f201713f932b58cb9de94a0cd524164a0f2283343547b3
    Status: Downloaded newer image for openjdk:8-jdk-alpine
     ---> a3562aa0b991
    Step 2/5 : VOLUME /tmp
     ---> Running in 22969c9e515d
    Removing intermediate container 22969c9e515d
     ---> adcaa2f47595
    Step 3/5 : ARG JAR_FILE
     ---> Running in 179a88cf7a54
    Removing intermediate container 179a88cf7a54
     ---> d941258bccbf
    Step 4/5 : COPY ${JAR_FILE} app.jar
     ---> 47dab086f113
    Step 5/5 : ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
     ---> Running in 4ae3ef13e76d
    Removing intermediate container 4ae3ef13e76d
     ---> c24e3c2bbce9
    Successfully built c24e3c2bbce9
   </pre>

3. Make sure your image is created as correctly:
   <pre>
   docker images
   </pre>
   should list at least the 2 images that we referenced, one is the openjdk image referenced by the `FROM` statement,
   the other is the newly created image based on the openjdk image:
   <pre>
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    demoapp             latest              12fd66d4f479        11 minutes ago      125MB
    openjdk             8-jdk-alpine        a3562aa0b991        4 months ago        105MB   
   </pre>
   
   If something isn't the way you expected it, you can delete the image with `docker rmi <image id>` and start over at the 
   previous step.
   
## Run the container

1. Running the container requires you to also expose the port of the demo app to the host. Usually, the port is bound to the
docker daemon and you need to tell it that you want it forwarded to your host machine. Do do this, run:
    <pre>
    docker run -p 8080:8080  demoapp
    </pre>

    You should now be able to connect to your app using e.g. `curl`:
    <pre>
    curl -X GET -H 'Content-Type:application/json' localhost:8080/hello
    </pre>
    
    Exercise:
    Try to make it work in `postman`.

## References
* [More Docker fun](https://www.katacoda.com/courses/container-runtimes)