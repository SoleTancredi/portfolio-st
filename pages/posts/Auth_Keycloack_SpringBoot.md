In this post, I will show how to use Keycloak in a Spring Boot application. Before we use Keycloak, we will cover some basics about what Keycloak is and why we use it.

To get started with this demo, you will need the following things:

-A Code Editor – IntelliJ
-Database – MySQL
-Keycloak
-Java 8

## What is Keycloak?

Keycloak is an open-source identity and access management solution for modern applications and services. Keycloak provides both SAML and OpenID protocol solutions.

## Why do we use Keycloak?

As mentioned, Keycloak provides identity and access management, it is also open source. SAML and OpenID protocols are industry standards. Building an application that is integrated with Keycloak will only provide you a more secure and stable solution. There are definitely other solutions available like Gluu, Shibboleth, WSO2.
For this post, we will be using Keycloak.

## Securing Spring Boot Application with Keycloak

There are two parts in this demo. One is about Keycloak. The second is about securing the Spring Boot Application with Keycloak.

## Install Keycloak

Download the [Keycloak]() on your machine. Unzip the downloaded file and run the server with the following command from bin directory on your command prompt (Note – I’m on a windows machine):

standalone.bat -Djboss.socket.binding.port-offset=100

This will start the Wildfly server for your Keycloak on your local machine. We can access the server by executing the URL http://localhost:8180. If you just use standalone.bat to execute without that parameter, the server will run on the port 8080.
## IMAGE

Once you start the server, the first thing you will have to do is to create an admin user. We will create a user admin and password d#n3q2b .

Now we will access the administration console and enter our user details. Once we login as an admin user, we will see the first screen as below:

## Adding Application

Initial screen shows the default realm. For our demo purposes, we will create a new realm SpringBootKeycloakApp. In this realm, we will add our Spring Boot application as a client. Create a new client on Clients tab. We will name our client application as SpringBootApp.

Now in settings, we will add redirect url for our Spring Boot Application. This is the URL where Keycloak will redirect to our app after authentication. Also, we are using openid connect as a protocol as part of this implementation.
## Adding User

Now we will add a user that we will use to authenticate. We will use this user to login to our sample Spring Boot application.

Add a role that you want for this user ROLE_User on the roles tab in Keycloak. Once that is done, let’s go to the Users tab and add a new user.

## Create A Spring Boot Application

Now, we will create a simple Spring Boot application that will use Keycloak for security. As part of this application, we will be showing a list of to-do list tasks for the user who will authenticate with the application.

To build this app, we need the following dependencies:

```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    implementation 'org.keycloak:keycloak-spring-boot-starter'
    runtimeOnly 'mysql:mysql-connector-java'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
    testImplementation 'org.springframework.security:spring-security-test'
}
```

As you can see we are using spring-boot and spring-security along with keycloak-spring-boot-starter dependency.

The keycloak dependency includes Keycloak client adapters. We will use these adapters for authentication purposes. They will replace our standard Spring Security adapters. To make sure this keycloak-spring-boot-starter dependency works correctly, we will need one more dependency to be added in our gradle file as below:

```
dependencyManagement {
    imports {
        mavenBom "org.keycloak.bom:keycloak-adapter-bom:11.0.2"
    }
}```

To read more about this, you can visit the official documentation of keycloak.

Our Controller class will have two important methods, one to get the home page which will be accessible for anyone, and another getting the list of tasks which will be accessible to only authenticated users with a role ROLE_User. The code for this TaskController will look like below:

```package com.betterjavacode.keycloakdemo.keycloakdemo.controllers;

import com.betterjavacode.keycloakdemo.keycloakdemo.dto.TaskDto;
import com.betterjavacode.keycloakdemo.keycloakdemo.managers.TaskManager;
import org.keycloak.KeycloakSecurityContext;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Controller
public class TaskController
{
    private final HttpServletRequest request;

    @Autowired
    public TaskController(HttpServletRequest request)
    {
        this.request = request;
    }

    @Autowired
    private TaskManager taskManager;

    @GetMapping(value="/")
    public String home()
    {
        return "index";
    }

    @GetMapping(value="/tasks")
    public String getTasks(Model model)
    {
        List tasks = taskManager.getAllTasks();
        model.addAttribute("tasks", tasks);
        model.addAttribute("name", getKeycloakSecurityContext().getIdToken().getGivenName());

        return "tasks";
    }

    private KeycloakSecurityContext getKeycloakSecurityContext()
    {
        return (KeycloakSecurityContext) request.getAttribute(KeycloakSecurityContext.class.getName());
    }

}
```

In this controller class, we use TaskManager to get all tasks. I will explain KeyCloakSecurityContext when I will show about SecurityConfig.

## With or without Spring-Security

We can leverage this application and use Keycloak for authentication with or without Spring-Security. As part of this demo, we are using Spring-Security. To use the same application without Spring-Security, you can just remove the Spring-Security dependency and add security configuration through application.properties file.

We will need the following properties in application.properties to use Keycloak for authentication in this app.
```keycloak.auth-server-url=http://localhost:8180/auth
keycloak.realm=SpringBootKeycloakApp
keycloak.resource=SpringBootApp
keycloak.public-client=true
keycloak.principal-attribute=preferred_username
```

If we wanted to use this application without Spring-Security, we will need the following two properties also:

```keycloak.security-constraints[0].authRoles[0]=ROLE_User
keycloak.security-constraints[0].securityCollections[0].patterns[0]=/tasks```

Since we are using Spring-Security, we will configure the security configuration through a Java class SecurityConfig.

This SecurityConfig class will extend KeyCloakWebSecurityConfigurerAdapter.

Our configure method will look like below:
```@Override
    protected void configure(HttpSecurity httpSecurity) throws Exception
    {
        super.configure(httpSecurity);
        httpSecurity.authorizeRequests()
                .antMatchers("/tasks").hasRole("User")
                .anyRequest().permitAll();
    }```
Basically any requests coming to /tasks endpoint, should have user role as ROLE_User. The prefix of ROLE_ is assumed here. Other than any other request will be permitted without any authorization. In this case, we will be calling our index page.

We will be using annotation @KeyCloakConfiguration which is basically covers @Configuration and @EnableWebSecurity annotations.

Since our SecurityConfig extends KeycloakWebSecurityConfigurerAdapter, we have to implement sessionAuthenticationStrategy and httpSessionManager. We will also have to register our idp Keycloak with Spring Security Authentication Manager.

So our SecurityConfig will look like below:
```package com.betterjavacode.keycloakdemo.keycloakdemo.config;

import org.keycloak.adapters.springsecurity.KeycloakConfiguration;
import org.keycloak.adapters.springsecurity.authentication.KeycloakAuthenticationProvider;
import org.keycloak.adapters.springsecurity.config.KeycloakWebSecurityConfigurerAdapter;
import org.keycloak.adapters.springsecurity.management.HttpSessionManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;

import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.authority.mapping.SimpleAuthorityMapper;
import org.springframework.security.core.session.SessionRegistryImpl;
import org.springframework.security.web.authentication.session.RegisterSessionAuthenticationStrategy;
import org.springframework.security.web.authentication.session.SessionAuthenticationStrategy;


@KeycloakConfiguration
public class SecurityConfig extends KeycloakWebSecurityConfigurerAdapter
{
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder authenticationManagerBuilder)
    {
        SimpleAuthorityMapper simpleAuthorityMapper = new SimpleAuthorityMapper();
        simpleAuthorityMapper.setPrefix("ROLE_");

        KeycloakAuthenticationProvider keycloakAuthenticationProvider =
                keycloakAuthenticationProvider();
        keycloakAuthenticationProvider.setGrantedAuthoritiesMapper(simpleAuthorityMapper);
        authenticationManagerBuilder.authenticationProvider(keycloakAuthenticationProvider);
    }

    @Bean
    @Override
    protected SessionAuthenticationStrategy sessionAuthenticationStrategy ()
    {
        return new RegisterSessionAuthenticationStrategy(new SessionRegistryImpl());
    }

    @Bean
    @Override
    @ConditionalOnMissingBean(HttpSessionManager.class)
    protected HttpSessionManager httpSessionManager()
    {
        return new HttpSessionManager();
    }

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception
    {
        super.configure(httpSecurity);
        httpSecurity.authorizeRequests()
                .antMatchers("/tasks").hasRole("User")
                .anyRequest().permitAll();
    }
}```
So Spring Security uses roles in upper case like ROLE_USER and always use ROLE_ prefix. To handle that, I have added a user with a role ROLE_User in Keycloak, but we will only verify a prefix as our http configuration will verify the role anyhow.

Since we will be authenticating with Keycloak, we will need a session for user’s state. We are using RegisterSessionAuthenticationStrategy here. HttpSessionManager is a conditional bean because Keycloak already implements that bean.

To implement Keycloak Spring Boot adapter, we will add a KeyCloakSpringBootConfigResolver bean as follows:
```package com.betterjavacode.keycloakdemo.keycloakdemo.config;

import org.keycloak.adapters.springboot.KeycloakSpringBootConfigResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class KeycloakConfig
{
    @Bean
    public KeycloakSpringBootConfigResolver keycloakSpringBootConfigResolver()
    {
        return new KeycloakSpringBootConfigResolver();
    }
}
```
## Demo of the application

Run our keycloak application, it will be running on http://localhost:8180. Our Spring Boot application will be running at http://localhost:8080.

Our first screen of the Spring Boot application will look like below:
#IMAGE
Now if a user clicks on Get all tasks, he will be redirected to Keycloak login screen as below:

#IMAGE

Now, I will enter my user betterjavacode username and password and it will show us our list of tasks as follows:

## Authentication Flow
When the user clicks on Get all tasks, the user is redirected to Spring Security’s sso/login endpoint which KeycloakSpringBootConfigResolver handles and sends an authorization code flow request to Keycloak

http://localhost:8180/auth/realms/SpringBootKeycloakApp/protocol/openid-connect/auth?response_type=code&client_id=SpringBootApp&redirect_uri=http%3A%2F%2Flocalhost%3A8080%2Fsso%2Flogin&state=70bd4e28-89e6-43b8-8bea-94c6d057a5cf&login=true&scope=openid

Keycloak will process the request to respond with a session code and show the login screen.

Once the user enters credentials and keycloak validates those, it will respond with an authorization code, and this code is exchanged for a token, and the user is logged in.

## Conclusion
In this post, I showed how to secure your Spring Boot application using Keycloak as an identity provider.