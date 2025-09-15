---
sidebar_position: 50
---

# Multiple Bean Instances

By default, SmartBeans creates a single instance of each bean. However, it also supports creating multiple instances of 
the same bean class when needed. Consider a scenario where you have a bean that controls a light with a motion sensor. 
If you need to control multiple lights with different motion sensors, it's more efficient to implement a single bean 
class and create separate instances for each light-sensor combination.

The `@SmartBeanDef` annotation facilitates this multi-instance capability. This annotation can be applied multiple times 
to the same bean class. Each annotation instance requires a unique `name` attribute that distinguishes it from other
instances of the same bean class. This name must be unique across all instances of that specific bean class.

````java
@SmartBeanDef(name = "KitchenMotion")
@SmartBeanDef(name = "BathroomMotion")
public class MotionControl implements SmartBean {

}
````

This example creates two distinct instances of the `MotionControl` bean class, one named "KitchenMotion" and one named 
"BathroomMotion". Creating multiple instances of the same bean without differentiation would have limited utility. 
However, in this scenario, each instance is intended to serve a specific purpose: controlling different lights based on 
input from different motion sensors. The "KitchenMotion" instance should respond to the kitchen motion sensor to control
the kitchen lighting, while the "BathroomMotion" instance should respond to the bathroom motion sensor to control the
bathroom lighting.

## Bean Parameters

Bean parameters can be used to configure each instance of a bean class. For example, if you want to configure the 
"KitchenMotion" instance to use a different motion sensor, you can do so by creating a `sensor` parameter and set it to the 
appropriate value. The same applies to the light you want to control. So our bean will have two parameters: `sensor` and 
`light`. To set these parameters, you can use the `params` attribute of the `@SmartBeanDef` annotation.

````java
@SmartBeanDef(name = "KitchenMotion", params = {
    @Param(name = "light", value = "light.kitchen_ceiling"),
    @Param(name = "sensor", value = "binary_sensor.kitchen_motion")
})
@SmartBeanDef(name = "BathroomMotion", params = {
    @Param(name = "light", value = "light.mirror_cabinet"),
    @Param(name = "sensor", value = "binary_sensor.bathroom_motion")
})
public class MotionControl implements SmartBean {

}
````

:::note
The name of a parameter must contain only alphanumeric characters, hyphens and underscores.
:::

### Accessing Bean Parameters

The simplest approach to access configuration parameters is to reference them directly within annotations in your bean 
class. This can be done, for instance, through an `@Entity` annotation on an instance field or a trigger annotation on 
a bean method. To retrieve the value of a parameter named `paramName`, use the expression syntax `${paramName}` within 
the annotation's attributes. This expression will be evaluated and replaced with the actual parameter value at runtime.

````java
@SmartBeanDef(name = "KitchenMotion", params = {
    @Param(name = "light", value = "light.kitchen_ceiling"),
    @Param(name = "sensor", value = "binary_sensor.kitchen_motion")
})
@SmartBeanDef(name = "BathroomMotion", params = {
    @Param(name = "light", value = "light.mirror_cabinet"),
    @Param(name = "sensor", value = "binary_sensor.bathroom_motion")
})
public class MotionControl implements SmartBean {

  @Entity("${light}")
  private Light light;

  @OnStateTrigger(entity = "${sensor}", to = "on")
  public void onMotion() {
    light.turnOn();
  }
}
````

You can also use parameters to construct a string as demonstrated in the following example.

````java
@SmartBeanDef(name = "KitchenMotion", params = {
    @Param(name = "room", value = "kitchen")
})
@SmartBeanDef(name = "BathroomMotion", params = {
    @Param(name = "room", value = "bathroom")
})
public class MotionControl implements SmartBean {

  @Entity("light.${room}_ceiling")
  private Light light;

  @OnStateTrigger(entity = "binary_sensor.${room}_motion", to = "on")
  public void onMotion() {
    light.turnOn();
  }
}
````

Another approach to parameter access is through the `SmartBeans` API's `getParameter(String)` method. This method 
returns a `BeanParameter` object that provides several utility methods for efficiently retrieving the parameter's value
with proper type handling.

````java
@SmartBeanDef(name = "KitchenMotion", params = {
    @Param(name = "room", value = "kitchen")
})
@SmartBeanDef(name = "BathroomMotion", params = {
    @Param(name = "room", value = "bathroom")
})
public class MotionControl implements SmartBean {

  private SmartBeans sb;

  @OnStateTrigger(entity = "binary_sensor.${room}_motion", to = "on")
  public void onMotion() {
    sb.log("Motion detected in the %s.", sb.getParameter("room").asString());
  }
}
````

### Using Default Values

Parameters can be made optional by specifying a default value that will be used when the parameter is not provided. This
is achieved using the syntax `${paramName|defaultValue}`, where the value after the pipe symbol serves as the fallback
value if the parameter is undefined or empty. The example below demonstrates this functionality.

````java
@SmartBeanDef(name = "KitchenMotion", params = {
    @Param(name = "lightOffTimeout", value = "00:00:30")
})
@SmartBeanDef(name = "BathroomMotion")
public class MotionControl implements SmartBean {

  @TimerDef(interval = @Interval("${lightOffTimeout|00:01:00}"))
  private Timer lightOffTimer;
}
````

The motion controller for the kitchen has a light-off timeout configured as 30 seconds (`00:00:30`). If no value is
explicitly provided for the `lightOffTimeout` parameter, as in the instance for the bathroom, the system will apply the
default value of 1 minute (`00:01:00`).
