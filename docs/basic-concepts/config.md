---
sidebar_position: 90
---

# Config entities

Similar to [provided entities](provided), SmartBeans can generate additional entities in Home Assistant to configure the 
bean. Config entities serve as input interfaces and are created in the `config` category of the bean device. SmartBeans 
supports various datatypes through different config entity types. For instance, the `ConfigNumber` interface enables 
users to input numerical values that can be accessed within your SmartBean implementation. In Home Assistant, this is 
represented as a `number` entity.

For example, in our sample bean for motion-controlled lighting, we need to make the duration configurable for automatically 
switching off the lights after motion is no longer detected. To implement this, we can create a `ConfigDuration` entity 
named `timeout`.

Similar to provided entities, you simply declare a field with the desired configuration type and annotate it with `@Config`. 
SmartBeans will then automatically register this entity in Home Assistant and inject an object into the field, allowing
your application to retrieve the user-configured value at runtime.

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light light;

  @Config(value = "60", unitOfMeasurement = "s", max = 600)
  private ConfigDuration timeout;

  @TimerDef
  private Timer lightOffTimer;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "off")
  public void onNoMotion() {
    lightOffTimer.start(timeout.toDuration());
  }

  @OnTimerElapsed("lightOffTimer")
  public void turnLightOff() {
    light.turnOff();
  }
}
````

The `value` property of the `@Config` annotation specifies the default (or initial) value for the configurable parameter 
and is the only required property of this annotation.

Various config entity types are available. Refer to the [Config entities reference](../reference/config) for comprehensive 
information about the different types and their implementation details.

## Light presets

One useful configuration entity that deserves special mention is the ability to configure light presets. If you need to 
specify the brightness and color of a light that is activated by the bean, you can utilize the `ConfigRgbColor` entity 
for this purpose.

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light light;

  @Config(value = "#FFFFFF")
  private ConfigRgbColor lightColor;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "on")
  public void onMotion() {
    lightColor.applyTo(light);
  }
}
````

Light preset configuration entities are exposed as light entities in Home Assistant. When you invoke the `applyTo()` 
method on a `ConfigRgbColor` object, both the color properties and brightness settings from the configuration entity will 
be transferred to the target light entity supplied as the method parameter.

## Entities in Home Assistant

All config entities created by the bean are created as MQTT entities in Home Assistant and are automatically registered to
the bean's device. The SmartBean device for this bean will look like this in Home Assistant:

![Fibonacci sequence in Home Assistant](/img/screenshots/config_entities.png)

## Persistent config values

Every config entity is automatically instantiated during bean deployment, with its value in Home Assistant initialized 
to the default value specified in the `@Config` annotation. Consequently, these configured values are not retained 
between redeployments, such as when SmartBeans or Home Assistant restarts.

Fortunately, all config interfaces in SmartBeans extend the `HasState` interface, enabling state persistence. See the
[bean state chapter](state) for comprehensive details. To ensure persistence of configuration values, you must apply 
the `@State` annotation to all fields that should be preserved across restarts.

````java
public class KitchenMotionControl implements SmartBean {

  @Config(value = "60", unitOfMeasurement = "s", max = 600, friendlyName = "Light timeout")
  @State
  private ConfigDuration timeout;

  @Config(value = "#FFFFFF", friendlyName = "Lightcolor")
  @State
  private ConfigRgbColor lightColor;
}
````
