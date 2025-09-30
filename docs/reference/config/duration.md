---
sidebar_position: 20
description: "A number representing a duration config parameter."
title: Duration
---

# Duration Config

The `ConfigDuration` can be used to create a configuration entity in Home Assistant for defining a duration. Technically, 
it represents a configurable duration value in the bean that can be controlled through a Home Assistant number entity. 
The value can be queried within your bean's code to influence its behavior. 

To create a configurable duration, declare a field of type `ConfigDuration` in your bean and annotate it with `@Config`. 
The annotation's attributes allow you to configure the details of the corresponding entity in Home Assistant. The `value`
attribute specifies the default or initial value and must be set to a valid `integer`. Unless otherwise specified, the
value is interpreted as the number of seconds for the duration.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {
  
  @Config(value = "30")
  private ConfigDuration lightOffTimeout;
  
  @TimerDef
  private Timer lightOffTimer;

  @OnStateTrigger(entity = "binary_sensor.kitchen_motion", to = "off")
  public void onNoMotion() {
    lightOffTimer.start(lightOffTimeout.toDuration());
  }
}
````

:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no config entities can be 
created, since they cannot be associated with a device.
:::

## Entity Configuration

The `@Config` annotation can be used to configure the number entity created for the configurable duration in detail. It
supports the following attributes:

| Attribute           | Description                                                                                                                                                                                                                                      |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `value`             | **Required:** The default or initial value of the configurable duration. Must be a valid `integer`. The `unitOfMeasurement` attribute defines the unit associated with this value.                                                               |
| `entityId`          | The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. If not set, SmartBeans generates an entity ID using the bean name and field name.                          |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                                                                                                                     |
| `icon`              | The icon displayed in Home Assistant for this entity.                                                                                                                                                                                            |
| `unitOfMeasurement` | The unit used for the number entity's value. Allowed options are: `ms` (milliseconds), `s` (seconds), `m` (minutes), or `h` (hours). Default is `s`.                                                                                             |
| `min`               | The minimum allowed value of the number entity. Default is `1`.                                                                                                                                                                                  |
| `max`               | The maximum allowed value of the number entity. Default is `100`.                                                                                                                                                                                |
| `step`              | The increment step for adjusting the number entity's value. Default is `1`.                                                                                                                                                                      |
| `displayMode`       | The display mode of the number entity in Home Assistant, for example `box` or `slider`. See the `mode` property of the [MQTT number entity](https://www.home-assistant.io/integrations/number.mqtt/) for all possible values. Default is `auto`. |
| `deviceClass`       | _Not used for durations._                                                                                                                                                                                                                        |
| `options`           | _Not used for durations._                                                                                                                                                                                                                        |
| `pattern`           | _Not used for durations._                                                                                                                                                                                                                        |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Query Current State

When the number entity's state changes in Home Assistant, the update is immediately propagated to SmartBeans. You can
query the current duration at any time by calling the appropriate methods on the `ConfigDuration` object. Several
convenience methods are provided, which return the value in different units and types, making it easy to integrate with 
other SmartBeans objects.

| Method                | Description                                                                                                                                                          |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `toMillis()`          | Returns the current value as the number of milliseconds.                                                                                                             |
| `toMillisPart()`      | Returns the millisecond part of the value, i.e., the remainder when dividing `toMillis()` by `1000`.                                                                 |
| `toSeconds()`         | Returns the current value as the number of seconds.                                                                                                                  |
| `toSecondsPart()`     | Returns the seconds part of the value, i.e., the remainder when dividing `toSeconds()` by `60`.                                                                      |
| `toMinutes()`         | Returns the current value as the number of minutes.                                                                                                                  |
| `toMinutesPart()`     | Returns the minutes part of the value, i.e., the remainder when dividing `toMinutes()` by `60`.                                                                      |
| `toHours()`           | Returns the current value as the number of hours.                                                                                                                    |
| `toDuration()`        | Returns the current value as a standard Java `Duration`.                                                                                                             |
| `toLightTransition()` | Returns the current value as a `LightAttr`, which can be used as a transition time in the `turnOn()` and `turnOff()` methods of a [light entity](../entities/light). |

Here is a complete example:

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {
  
  @Config(
      value = "300",
      unitOfMeasurement = "ms",
      min = 0,
      max = 2000,
      step = 50,
      friendlyName = "Light transition time",
      icon = "mdi:timer"
  )
  private ConfigDuration transitionTime;
  
  @Entity("light.kitchen_ceiling")
  private Light ceilingLight;

  @OnStateTrigger(entity = "binary_sensor.kitchen_motion", to = "on")
  public void onMotion() {
    ceilingLight.turnOn(transitionTime.toLightTransition());
  }
}
````

## Create Entity Programmatically

In addition to the annotation-based approach, you can programmatically create a configurable duration using the 
`getConfigDuration()` method of the `SmartBeans` API. This approach is useful in some edge cases when the entity's
attributes are generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the configuration name, which must be unique within the scope of the
bean; the second is the default or initial value; and the third is a builder for defining all other attributes of the 
entity in Home Assistant.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {
  
  private ConfigDuration transitionTime;

  @Override
  public void init(SmartBeans sb) {
    transitionTime = sb.getConfigDuration("transitionTime", Duration.ofMillis(300), def -> def
        .setUnitOfMeasurement(DurationUnit.MILLISECONDS)
        .setMin(Duration.ZERO)
        .setMax(Duration.ofMillis(2000))
        .setStep(Duration.ofMillis(50))
        .setFriendlyName("Light transition time")
        .setIcon("mdi:timer")
        .setDisplayMode(DisplayMode.SLIDER)
    );
  }
}
````
