# Quick Start Guide

This guide walks you through setting up SmartBeans and creating your first SmartBean.  
The process consists of a few simple steps:

1. Install the SmartBeans add-on in Home Assistant.  
2. Set up a Java project with Maven to implement your beans.
3. Create your bean.
4. Build a JAR file containing your beans.  
5. Deploy the JAR file to your SmartBeans installation in Home Assistant.  

### Prerequisites

Before you begin, make sure you have the following:

- Home Assistant running with a installation method, that supports [add-ons](https://www.home-assistant.io/addons), like HA OS.
- A locally installed Java Development Kit 17 (or higher) on the machine where you will develop your beans.  
  For example: [OpenJDK](https://openjdk.org/projects/jdk/17/).  
- [Apache Maven](https://maven.apache.org/) installed and properly configured.  
- A Java IDE of your choice, such as [IntelliJ IDEA](https://www.jetbrains.com/idea/).  
- Access to your Home Assistant configuration directory in order to copy files. This can be done for example via the FTP
  add-on or using SCP.  

:::note
You don’t have to use Maven or IntelliJ IDEA to implement your beans. Any build tool (e.g., Gradle) and any IDE
(e.g., Eclipse) will work - or even a plain text editor with manual compilation using `javac`. This guide uses Maven
because it is widely known and provides a straightforward setup.  
:::

:::note
Running SmartBeans as a Home Assistant add-on is the easiest way to get started. If your Home Assistant installation 
does not support add-ons, you can still run SmartBeans as a standalone Java program. However, the installation, 
configuration, and onboarding process is significantly more complex in that case.  
:::

## Install SmartBeans Add-on

To install the SmartBeans add-on in Home Assistant, open the add-on store of your installation:

[![Open your Home Assistant instance and show the add-on store.](https://my.home-assistant.io/badges/supervisor_store.svg)](https://my.home-assistant.io/redirect/supervisor_store/)

Alternatively, navigate manually: _Settings_ → _Add-ons_ → _Add-on store_.

From the menu in the upper right (three dots), choose _Repositories_ and add the SmartBeans add-on repository by entering:  
`https://github.com/beanssmart/addon-repository` and clicking _Add_.

The SmartBeans repository should now appear in the list of available add-ons (refresh the page if it doesn’t).  
Click on the SmartBeans add-on and press _Install_.

The add-on includes many configuration options, but you typically don’t need to adjust them. We'll cover these options
later when customization becomes relevant.

Finally, press _Start_ to launch the add-on and check the log to confirm it started successfully.  
If you see the message  
`...successfully connected to HA`  
everything is working correctly.

## Setting up the Java Project

Open your IDE of choice and create a new Java Maven project. In IntelliJ IDEA this is done via _File_ → _New_ → _Project..._.

In the following dialog, select **Java** as the project type and **Maven** as the build system. Choose a project name and ensure **JDK 17** is selected.

After the project is created, open the `pom.xml`. It should look similar to this:

````xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>smartbeans-demo</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

</project>
````

The values for `groupId`, `artifactId`, and `version` are arbitrary and can be freely chosen.

:::note
You may use a newer JDK if you prefer, but it is essential that the `maven.compiler.source` and `maven.compiler.target` 
properties remain set to `17`.
:::

Now add the **SmartBeans API** as a Maven dependency. Make sure to check the latest SmartBeans version and adjust the 
value of the `smartbeans-version` property accordingly. It is important to use the same version of SmartBeans here as 
the SmartBeans add-on running in Home Assistant to avoid compatibility issues.

````xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>smartbeans-demo</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

    <smartbeans-version>1.0.0-beta-1</smartbeans-version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>io.github.beanssmart</groupId>
      <artifactId>smartbeans-api</artifactId>
      <version>${smartbeans-version}</version>
    </dependency>
  </dependencies>

</project>
````

After updating the `pom.xml`, you need to **sync the Maven changes**. In IntelliJ IDEA this is done by clicking the
_Maven update_ button in the top right corner of the editor, or by pressing `Alt+F5`. Maven will then automatically 
download all required JARs and place them in your local Maven repository.

## Create your First SmartBean

Now it’s time to create your first SmartBean. Start by creating a package of your choice in the `src/main/java` directory
of your Maven project, for example `com.example`.

In IntelliJ IDEA: Right-click on the `java` folder inside `src/main/java`, choose _New_ → _Package_, enter `com.example`, 
and press **Enter**.

Next, create a Java class inside this package: Right-click the newly created package, choose _New_ → _Java Class_, enter
a class name, and confirm with **Enter**.  

You are now ready to implement your first SmartBean - see the example below.

````java
package com.example;

import io.github.beanssmart.smartbeans.api.*;

public class HelloWorldBean implements SmartBean {
  @Override
  public void init(SmartBeans sb) {
    sb.log("Hello SmartBeans!");
  }
}
````
This first example SmartBean does nothing except writing **"Hello SmartBeans!"** into the log when it is deployed.  

## Build JAR File

To deploy your beans to SmartBeans in Home Assistant, you need to compile all Java classes and package them into a JAR file.  
With Maven this is very simple: just run `mvn package` in the root directory of your project.

In IntelliJ IDEA you can open a terminal with `Alt+F12` or via _View_ → _Tool Windows_ → _Terminal_. Make sure the 
terminal is in the same directory as your `pom.xml`, then enter:

````bash
mvn package
````

Maven will compile all classes and package them into a single JAR file. The resulting JAR can be found in the target
directory. Its name is based on the `artifactId` and `version` you defined in the `pom.xml`, for example 
`smartbeans-demo-1.0-SNAPSHOT.jar`.

## Deploy JAR File to Add-on

To deploy your bean to SmartBeans, copy the created JAR file to the configuration directory of your Home Assistant 
installation (the same directory where the `configuration.yaml` file is located).  

Create a subdirectory named `smartbeans` inside the configuration directory and place your JAR file there. For example,
you can use the FTP add-on in Home Assistant to transfer the file.

:::info
If you deploy your first bean and the `smartbeans` directory did not exist before you started the SmartBeans add-on, 
you must **restart the add-on after creating the directory**. From then on, SmartBeans continuously watches this 
directory for file changes. Whenever a JAR is added, updated, or deleted, the corresponding beans are automatically 
deployed, redeployed, or undeployed.  

If the directory was missing at startup, you will see a warning in the logs indicating that SmartBeans could not install 
the file watcher.
````
WARN  [main]: Bean lib dir does not exist: /homeassistant_config/smartbeans
````
:::

When you deploy the bean successfully, you should see the following message in the logs:
````
INFO  [HelloWorldBean]: Hello SmartBeans!
````

You are now all set and ready to start creating your first automation with SmartBeans.
