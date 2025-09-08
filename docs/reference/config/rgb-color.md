---
sidebar_position: 20
description: "A light entity representing a preset defined by values of red, green and blue."
title: RGB Color
---

# RGB color config

The `ConfigRgbColor` can be used to create a light entity in Home Assistant for configuring a light preset based on the 
red, green, and blue components of the light color. Technically, it represents a configurable light preset in the bean 
that can be controlled through a Home Assistant light entity. The current preset can be queried within your bean's code 
and applied to real lights. 

To create an RGB-based light preset, declare a field of type `ConfigRgbColor` in your bean and annotate it with 
`@Config`. The annotation's attributes allow you to configure the details of the corresponding entity in Home Assistant. 
The `value` attribute specifies the default or initial value and must be set to three comma-separated `integer` values
representing the red, green, and blue components of the light color.
 
````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light ceiling;

  @Config(value = "255,120,150")
  private ConfigRgbColor lightColor;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "on")
  public void onMotion() {
    lightColor.applyTo(ceiling);
  }
}
````

:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no config entities can be 
created, since they cannot be associated with a device.
:::

## Entity configuration

The `@Config` annotation can be used to configure the light entity created for the light preset in detail. It
supports the following attributes:

| Attribute           | Description                                                                                                                                                                                                                              |
|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `value`             | **Required:** The default or inital value of the configurable boolean, must be three comma-separated `integer` values representing red, green and blue components of the color.                                                          |
| `entityId`          | The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. If not set SmartBeans generates an entity ID using the name of the bean and the name of the field. |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                                                                                                             |
| `icon`              | The icon shown in Home Assistant for this entity.                                                                                                                                                                                        |
| `deviceClass`       | _Not used for RGB color configs._                                                                                                                                                                                                        |
| `unitOfMeasurement` | _Not used for RGB color configs._                                                                                                                                                                                                        |
| `min`               | _Not used for RGB color configs._                                                                                                                                                                                                        |
| `max`               | _Not used for RGB color configs._                                                                                                                                                                                                        |
| `step`              | _Not used for RGB color configs._                                                                                                                                                                                                        |
| `options`           | _Not used for RGB color configs._                                                                                                                                                                                                        |
| `displayMode`       | _Not used for RGB color configs._                                                                                                                                                                                                        |
| `pattern`           | _Not used for RGB color configs._                                                                                                                                                                                                        |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Using current preset

When the light entity's color or brightness changes in Home Assistant, the update is immediately propagated to SmartBeans. 
You can query the current preset at any time by calling the appropriate methods on the `ConfigRgbColor` object. In 
addition to standard getters for retrieving the value, there are convenient `applyTo()` methods that allow you to directly
apply the current preset to a light, as shown in the example above.

| Method                           | Description                                                                                                                                          |
|----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| `getRgbColor()`                  | Returns the current color represented by a `RgbColor` object containing values for red, green and blue components of the color.                      |
| `getBrightness()`                | Returns the current brightness value between 0 and 255 as an `integer`.                                                                              |
| `applyTo(Light)`                 | Applies the current light preset to the specified light entity.                                                                                      |
| `applyTo(Light, double)`         | Applies the current light preset to the specified light entity with the given transition time in seconds.                                            |
| `applyTo(Light, ConfigDuration)` | Applies the current light preset to the specified light entity with the given transition time, represented by a [configurable duration](./duration). |

Here is a complete example:

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light ceiling;

  @Config(
      value = "120,80,180",
      entityId = "light.config_lightcolor_kitchen_motion",
      friendlyName = "Light color on motion",
      icon = "mdi:palette"
  )
  private ConfigRgbColor lightColor;

  @OnStateTrigger(entity = "binary_sensor.motion_kitchen", to = "on")
  public void onMotion() {
    lightColor.applyTo(ceiling, 0.5);
  }
}
````

## Create entity programmatically

In addition to the annotation-based approach, you can programmatically create a RGB-based light preset 
using the `getConfigRgbColor()` method of the `SmartBeans` API. This approach is useful in some edge cases when the
entity's attributes are generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the configuration name, which must be unique within the scope of the
bean; the second is the default or initial value; and the third is a builder for defining all other attributes of the 
entity in Home Assistant.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {

  private ConfigRgbColor lightColor;

  @Override
  public void init(SmartBeans sb) {
    lightColor = sb.getConfigRgbColor("lightcolor", new Light.RgbColor(140, 200, 40), def -> def
        .setFriendlyName("Light color on motion")
        .setEntityId("light.config_lightcolor_kitchen_motion")
        .setIcon("mdi:palette")
    );
  }
}
````
