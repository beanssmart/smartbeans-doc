---
sidebar_position: 20
description: "A switch representing a boolean config parameter."
title: Boolean
---

# Boolean Config

The `ConfigBoolean` can be used to create a configuration switch in Home Assistant to enable or disable a feature of the
bean. Technically, it represents a configurable `boolean` value in the bean that can be controlled through a Home
Assistant switch entity. The value can be queried within your bean's code to influence its behavior. 

To create a configurable boolean, declare a field of type `ConfigBoolean` in your bean and annotate it with `@Config`. 
The annotation's attributes allow you to configure the details of the corresponding entity in Home Assistant. The 
`value` attribute specifies the default or initial value of the configurable boolean and must be set to either `true` 
or `false`.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  @Config(value = "false")
  private ConfigBoolean aOption;

  public void someBeanMethod() {
    if(aOption.isSet()) {
      //do stuff, only if option is set by user.
    }
  }
}
````

:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no config entities can be 
created, since they cannot be associated with a device.
:::

## Entity Configuration

The `@Config` annotation can be used to configure the switch entity created for the configurable boolean in detail. It
supports the following attributes:

| Attribute           | Description                                                                                                                                                                                                                              |
|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `value`             | **Required:** The default or inital value of the configurable boolean, must be either `true` or `false`                                                                                                                                  |
| `entityId`          | The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. If not set SmartBeans generates an entity ID using the name of the bean and the name of the field. |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                                                                                                             |
| `icon`              | The icon shown in Home Assistant for this entity. If no icon is specified, it is derived from the device class.                                                                                                                          |
| `deviceClass`       | The [device class](https://www.home-assistant.io/integrations/switch/#device-class) of the switch entity in Home Assistant.                                                                                                              |
| `unitOfMeasurement` | _Not used for booleans._                                                                                                                                                                                                                 |
| `min`               | _Not used for booleans._                                                                                                                                                                                                                 |
| `max`               | _Not used for booleans._                                                                                                                                                                                                                 |
| `step`              | _Not used for booleans._                                                                                                                                                                                                                 |
| `options`           | _Not used for booleans._                                                                                                                                                                                                                 |
| `displayMode`       | _Not used for booleans._                                                                                                                                                                                                                 |
| `pattern`           | _Not used for booleans._                                                                                                                                                                                                                 |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Query Current State

When the switch entity's state changes in Home Assistant, it is immediately propagated to SmartBeans. You can query the
current state at any time by calling the appropriate methods on the `ConfigBoolean` object. Several convenience methods
are provided, which serve the same purpose but differ in naming to improve code readability.

| Method       | Description                                                                           |
|--------------|---------------------------------------------------------------------------------------|
| `getValue()` | Returns the current value as `boolean`, `true` when the state of the switch is `on`.  |
| `isTrue()`   | Returns the current value as `boolean`, `true` when the state of the switch is `on`.  |
| `isFalse()`  | Returns the current value as `boolean`, `true` when the state of the switch is `off`. |
| `isSet()`    | Returns the current value as `boolean`, `true` when the state of the switch is `on`.  |

Here is a complete example:

````java
@SmartBeanDef(beanDevice = @BeanDevice(type = BeanDevice.EntityType.SWITCH))
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light ceiling;

  @Config(
      value = "true",
      friendlyName = "Turn light off",
      icon = "mdi:lightbulb-off-outline"
  )
  private ConfigBoolean turnOffByMotion;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "off")
  public void turnLightOff() {
    if(turnOffByMotion.isTrue()) {
      ceiling.turnOff();
    }
  }
}
````

:::note
You don't need to create a config entity to enable or disable an entire bean. This functionality is built into SmartBeans 
and can be enabled by specifying `@BeanDevice(type = BeanDevice.EntityType.SWITCH)` as the `beanDevice` in the 
`@SmartBeanDef` annotation. This creates a switch in Home Assistant that controls the entire bean, including all methods, 
triggers, and timers.
:::

## Create Entity Programmatically

In addition to the annotation-based approach, you can programmatically create a configurable boolean using the 
`getConfigBoolean()` method of the `SmartBeans` API. This approach is useful in some edge cases when the entity's
attributes are generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the configuration name, which must be unique within the scope of the
bean; the second is the default or initial value; and the third is a builder for defining all other attributes of the 
entity in Home Assistant.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {

  private ConfigBoolean turnOffByMotion;

  @Override
  public void init(SmartBeans sb) {
    turnOffByMotion = sb.getConfigBoolean("turnOffByMotion", true, def -> def
        .setFriendlyName("Turn light off")
        .setIcon("mdi:lightbulb-off-outline")
    );
  }
}
````
