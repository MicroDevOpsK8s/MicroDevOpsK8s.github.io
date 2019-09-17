---
layout: lab
title: Extend the sample code
---

In this lab you will extend the existing code to make it a REST service. In order to do that, we'll use SpringBoot annotations.

# Create a new class

Depending on your editor, you'll need to either create a new class or a new file in a certain subdirectory.

1. Open the `demo/src/main/kotlin/com.example.demo` folder in your editor. There you'll find a class named `DemoApplication`.

1. Either do the following:
- In editors that allow you to create a new class, create a new class named: `com.example.demo.ctrl.DemoController`.
- In other editors, create a new directory called `demo/src/main/kotlin/com/example/demo/ctrl` and create new file named `DemoController.kt`.

1. Add the following code to your newly created file or class:

    <pre>

     package com.myexample.demo.ctrl

     import org.springframework.web.bind.annotation.*;
     import org.springframework.stereotype.Controller
     
     @RestController
     @RequestMapping("/hello")
     class DemoController {
     
       @GetMapping
       fun hello(): String {
          return "Hello World!"
       }
    }
    </pre>

2. Let's take a closer look to the code: `package` and `import` should be somewhat clear, so let's talk about the class 
and the Annotations. The class is called `DemoController` and is preceded by 2 annotations named `@RestController` and 
`@RequestMapping(...)`. These annotations firstly create a RestController: It opens a server socket on port 8080 
(because it's the default in the Spring framework), then adds a handler for the url `hello`, so that you finally can call
your service.

# Build your code

2. Open a command line in your code directory and run `gradle build`. This should succeed with a message similar to:
    <pre>
    $ gradle build
    
    > Task :test
    2019-09-16 15:13:13.795  INFO 8681 --- [       Thread-4] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
    
    Deprecated Gradle features were used in this build, making it incompatible with Gradle 6.0.
    Use '--warning-mode all' to show the individual deprecation warnings.
    See https://docs.gradle.org/5.5.1/userguide/command_line_interface.html#sec:command_line_warnings
    
    BUILD SUCCESSFUL in 5s
    6 actionable tasks: 6 executed
    </pre>


# Run your code

The build step should have created a file named `demo/build/libs/demo-0.0.1-SNAPSHOT.jar`. So now, we can execute the rest service
with:

`java -jar build/libs/demo-0.0.1.SNAPSHOT.jar`

The output generated looks similar to this:
<pre>
$ java -jar build/libs/demo-0.0.1-SNAPSHOT.jar 

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.8.RELEASE)

2019-09-16 15:18:27.575  INFO 8714 --- [           main] com.myexample.demo.DemoApplicationKt     : Starting DemoApplicationKt on Matthiass-MacBook-Pro.local with PID 8714 (/Users/matthiaskubik/dev/azure/demo/build/libs/demo-0.0.1-SNAPSHOT.jar started by matthiaskubik in /Users/matthiaskubik/dev/azure/demo)
2019-09-16 15:18:27.577  INFO 8714 --- [           main] com.myexample.demo.DemoApplicationKt     : No active profile set, falling back to default profiles: default
2019-09-16 15:18:28.383  INFO 8714 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-09-16 15:18:28.411  INFO 8714 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-09-16 15:18:28.411  INFO 8714 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.24]
2019-09-16 15:18:28.483  INFO 8714 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-09-16 15:18:28.483  INFO 8714 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 863 ms
2019-09-16 15:18:28.683  INFO 8714 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-09-16 15:18:28.860  INFO 8714 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-09-16 15:18:28.864  INFO 8714 --- [           main] com.myexample.demo.DemoApplicationKt     : Started DemoApplicationKt in 1.919 seconds (JVM running for 2.219)
    
</pre> 

Take some time to see what happens.

## References
* [SpringBoot Introduction](https://www.youtube.com/playlist?list=PLqq-6Pq4lTTbx8p2oCgcAQGQyqN8XeA1x)