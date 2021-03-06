# microservices-config
Configurations of microservices architecture - Centralized point to keep important files.


# Centralized Configuration

https://spring.io/guides/gs/centralized-configuration/

http://cloud.spring.io/spring-cloud-config/spring-cloud-config.html#_quick_start


--------------------------------------------------------------------------------------------------------


#Stand up a Config Server

You’ll first need a Config Service to act as a sort of intermediary between your Spring applications and a typically version-controlled repository of configuration files. You can use Spring Cloud’s @EnableConfigServer to standup a config server that other applications can talk to. This is a regular Spring Boot application with one annotation added to enable the config server.


configuration-service/src/main/java/hello/ConfigServiceApplication.java


The Config Server needs to know which repository to manage. There are several choices here, but we’ll use a Git-based filesystem repository. You could as easily point the Config Server to a Github or GitLab repository, as well. On the file system, create a new directory and git init it. Then add a file called a-bootiful-client.properties to the Git repository. Make sure to also git commit it, as well. Later, you will connect to the Config Server with a Spring Boot application whose spring.application.name property identifies it as a-bootiful-client to the Config Server. This is how the Config Server will know which set of configuration to send to a specific client. It will also send all the values from any file named application.properties or application.yml in the Git repository. Property keys in more specifically named files (like a-bootiful-client.properties) override those in application.properties or application.yml.


Add a simple property and value, message = Hello world, to the newly created a-bootiful-client.properties file and then git commit the change.


Specify the path to the Git repository by specifying the spring.cloud.config.server.git.uri property in configuration-service/src/main/resources/application.properties. Make sure to also specify a different server.port value to avoid port conflicts when you run both this server and another Spring Boot application on the same machine.

configuration-service/src/main/resources/application.properties


#Reading Configuration from the Config Server using the Config Client

Now that we’ve stood up a Config Server, let’s stand up a new Spring Boot application that uses the Config Server to load its own configuration and that refreshes its configuration to reflect changes to the Config Server on-demand, without restarting the JVM. Add the org.springframework.cloud:spring-cloud-starter-config dependency in order to connect to the Config Server. Spring will see the configuration property files just like it would any property file loaded from application.properties or application.yml or any other PropertySource.

The properties to configure the Config Client must necessarily be read in before the rest of the application’s configuration is read from the Config Server, during the bootstrap phase. Specify the client’s spring.application.name as a-bootiful-client and the location of the Config Server spring.cloud.config.uri in configuration-client/src/main/resources/bootstrap.properties, where it will be loaded earlier than any other configuration.

configuration-client/src/main/resources/bootstrap.properties

The client may access any value in the Config Server using the traditional mechanisms (e.g. @ConfigurationProperties, @Value("${…​}") or through the Environment abstraction). Create a Spring MVC REST controller that returns the resolved message property’s value. Consult the Building a RESTful Web Service guide to learn more about building REST services with Spring MVC and Spring Boot.

By default, the configuration values are read on the client’s startup, and not again. You can force a bean to refresh its configuration - to pull updated values from the Config Server - by annotating the MessageRestController with the Spring Cloud Config @RefreshScope and then by triggering a refresh event.

configuration-client/src/main/java/hello/ConfigClientApplication.java

#Test the application

Test the end-to-end result by starting the Config Service first and then, once loaded, starting the client. Visit the client app in the browser, http://localhost:8080/message. There, you should see the String Hello world reflected in the response.

Change the message key in the a-bootiful-client.properties file in the Git repository to something different (Hello Spring!, perhaps?). You can confirm that the Config Server sees the change by visiting http://localhost:8888/a-bootiful-client/default. You need to invoke the refresh Spring Boot Actuator endpoint in order to force the client to refresh itself and draw the new value in. Spring 's Actuator exposes operational endpoints, like health checks and environment information, about an application. In order to use it you must add org.springframework.boot:spring-boot-starter-actuator to the client app’s CLASSPATH. You can invoke the refresh Actuator endpoint by sending an empty HTTP POST to the client’s refresh endpoint, http://localhost:8080/refresh, and then confirm it worked by reviewing the http://localhost:8080/message endpoint.

#Summary

Congratulations! You’ve just used Spring to centralize configuration for all your services by first standing up a and to then dynamically update configuration.

https://raw.githubusercontent.com/spring-guides/getting-started-macros/master/footer.adoc


-------------------------------------------------------------------------------------------------------------
# SVN configuration
server:
  port: 8888

spring:

  cloud:
  
    config:
    
      server:
      
        svn:
      
      uri: https://subversion.assembla.com/svn/spring-cloud-config-repo/
      
      #git:
        #  uri: https://github.com/pcf-guides/configuration-server-config-repo
      
      default-label: trunk
  
  profiles:
  
  active: subversion

logging:

levels:

        org.springframework.boot.env.PropertySourcesLoader: TRACE
    
       org.springframework.cloud.config.server: DEBUG
       
       
       ----------------------------------------------------------------------------
       
       https://dzone.com/articles/spring-31-environment-profiles
       
       
