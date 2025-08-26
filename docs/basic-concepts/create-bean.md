---
sidebar_position: 10
---

# Create a bean class

Creating a SmartBean is straightforward: simply create a Java class that implements the `SmartBean` interface.
SmartBeans automatically detects and deploys all classes implementing this interface during runtime by scanning both the
classpath and specified filesystem locations.

````java
public class MotionLightControl implements SmartBean {

  private SmartBeans sb;

}
````

The `sb` object in this class is an instance of SmartBeans and represents the SmartBeans context, providing access to 
the framework's API. The framework automatically injects this object at runtime, eliminating the need for manual 
initialization. Every SmartBean requires a public default constructor - which is automatically available if you don't 
define any constructors explicitly.

# Lifecycle

Each SmartBean implements two lifecycle methods that come with an empty default implementation but can be overridden 
when needed. 

The `init()` method is called when the bean is deployed and activated, while the `destroy()` method is invoked upon 
deactivation (for example, during SmartBeans shutdown). Both methods accept a `SmartBeans` instance as their parameter.

````java
public class MotionLightControl implements SmartBean {

  @Override
  public void init(SmartBeans sb) {
    sb.log("initializing motion control...");
  }

  @Override
  public void destroy(SmartBeans sb) {
    sb.log("motion control stopped, cleaning up...");
  }
}
````

# The `SmartBeans` API Interface

The `SmartBeans` object serves as the central programming interface for a SmartBean. While annotations cover most common
use cases, the `SmartBeans` object provides direct API access for advanced scenarios. This interface allows you to
programmatically interact with any Home Assistant service or trigger, even those not explicitly supported by the
framework's annotations.

The most common methods at `SmartBeans` are:

| Method                               | Description                                                                              |
|--------------------------------------|------------------------------------------------------------------------------------------|
| `log(String)`                        | Writes a message to the bean's dedicated log file for monitoring and debugging purposes. |
| `getBean(Class[, String])`           | Retrieves other SmartBean instances to enable direct method invocation between beans.    |
| `getNow()`                           | Returns the current local time. Preferred over `Instant.now()` for consistency.          |
| `callService(Service)`               | Calls any Home Assistant service (also known as action).                                 |
| `registerTrigger(TriggerDefinition)` | Programmatically subscribes to any Home Assistant trigger event.                         |

There are a lot more methods, but normally you do not need to use them. For example you can access a light entity by
calling `getLight("entityId")` but it is easier to use an annotation. For a complete reference check out the 
[SmartBeans API Reference](../reference/smart-beans-api).