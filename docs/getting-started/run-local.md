---
sidebar_position: 20
---

# Local Runner

When developing a new bean or modifying an existing one, it's convenient to test it locally before deploying to your 
Home Assistant server. SmartBeans allows you to run a single bean locally and connect it to your Home Assistant instance. 
This enables you to test (or even debug) your bean in a real smart home environment without continuously rebuilding and 
redeploying the JAR. Simply start the bean from your IDE and test it.

All you need is:

- The address and port of your Home Assistant instance.
- A long-lived access token for authentication.

## Create a Long-Lived Access Token

To create a long-lived access token, go to your user profile security settings or use this link:

[![Open your Home Assistant instance and show your Home Assistant user's security options.](https://my.home-assistant.io/badges/profile_security.svg)](https://my.home-assistant.io/redirect/profile_security/)

At the bottom of the page, in the _Long-Lived Access Tokens_ section, click _Create Token_. Enter a name for the token, 
click _OK_, and copy the generated token to a safe place.

## Start Bean Locally

To run your bean locally, you first need the SmartBeans runtime environment as a dependency in your project. To do this, 
edit your `pom.xml` file and add the following dependency:

````xml
<?xml version="1.0" encoding="UTF-8"?>
<project ...>
  
  ...

  <dependencies>
    ...

    <dependency>
      <groupId>io.github.beanssmart</groupId>
      <artifactId>smartbeans-local</artifactId>
      <version>${smartbeans-version}</version>
    </dependency>
  
  </dependencies>

</project>
````

Now you can create a simple `main()` method in your bean class to start the bean locally.

````java
public class HelloWorldBean implements SmartBean {
  @Override
  public void init(SmartBeans sb) {
    sb.log("Hello SmartBeans!");
  }

  public static void main(String[] args) {
    String host = "homeassistant.local"; //Home Assistant IP or hostname
    String token = "..."; //created long-lived access token
    LocalRunner runner = new LocalRunner(host, token);
    runner.runBean(HelloWorldBean.class);
  }
}
````

When you start this class, you should see an output similar to the following:

````
WARN  [main]: LocalRunner is for testing only, do not use in production environment!
INFO  [main]: Try to reconnect to HA...
INFO  [main]: Connected to websocket at ws://homeassistant.local:8123/api/websocket
INFO  [main]: ...successfully connected to HA.
INFO  [main]: Deploying bean com.example.HelloWorldBean
INFO  [main]: Loading bean state com.example.HelloWorldBean from: {...}/com.example.HelloWorldBean.json
INFO  [HelloWorldBean]: Hello SmartBeans!
INFO  [main]: Bean com.example.HelloWorldBean successfully deployed.
````

### Keeping the Classpath Clean

Running your bean locally with the runtime dependency works fine in general. However, there is a potential pitfall:  
because the runtime dependency is now on the classpath, you might accidentally use internal classes from the SmartBeans
runtime in your bean. While this may not be an issue immediately, it can cause problems later if you upgrade to a new
SmartBeans version where internals have changed.  

To avoid this risk, it is strongly recommended to rely only on the SmartBeans **API module** in your production code.  

A good practice is to restrict the runtime dependency to the **test scope** in Maven. Then, you can create a separate
class with a `main()` method in the `src/test/java` directory of your project. This way, internal classes remain
inaccessible to your actual bean code, while still allowing you to run and debug your beans locally.

````xml
    <dependency>
      <groupId>io.github.beanssmart</groupId>
      <artifactId>smartbeans-local</artifactId>
      <version>${smartbeans-version}</version>
      <scope>test</scope>
    </dependency>
````

````java
public class TestBean {

  public static void main(String[] args) {
    String host = "homeassistant.local";
    String token = "...";
    LocalRunner runner = new LocalRunner(host, token);
    runner.runBean(HelloWorldBean.class);
  }
}
````

An even stricter approach is to create a **separate project** dedicated to testing your beans. With this setup, even 
your test classes cannot accidentally reference internal runtime classes.  

Another benefit: your long-lived access token can remain outside your main project. This reduces the risk of 
accidentally committing sensitive credentials into version control (e.g., Git) or exposing them when sharing your code
with others.  
