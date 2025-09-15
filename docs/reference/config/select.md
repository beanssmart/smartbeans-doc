---
sidebar_position: 20
description: "A select representing a config parameter with fixed values (enum)."
title: Select
---

# Select Config

The `ConfigSelect` can be used to create a configuration entity in Home Assistant for selecting a value from a fixed set, 
similar to a Java enum. Technically, it represents a configurable value in the bean that can be controlled through a Home
Assistant select entity. The current value can be queried within your bean's code to influence its behavior.

To create a configurable value with a fixed value set, declare a field of type `ConfigSelect` in your bean and annotate 
it with `@Config`. The annotation's attributes allow you to configure the details of the corresponding entity in Home 
Assistant. The `value` attribute specifies the default or initial value and the `options` attribute is used to define 
all possible values.  

In Home Assistant, a select entity always consists of `String` values, but SmartBeans also supports creating selects 
based on a Java enum or even any other Java type, as we will see later.  

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {
  
  @Config(value = "white", options = {"white", "red", "yellow", "pink"})
  private ConfigSelect<String> lightColor;
  
  @Entity("light.kitchen_ceiling")
  private Light ceiling;

  public void onMotion() {
    ceiling.turnOn(LightAttr.colorName(lightColor.getValue()));
  }
}
````

:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no config entities can be 
created, since they cannot be associated with a device.
:::

## Entity Configuration

The `@Config` annotation can be used to configure the select entity created for the configurable value in detail. It
supports the following attributes:

| Attribute           | Description                                                                                                                                                                                                             |
|---------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `value`             | **Required:** The default or initial value of the configurable duration. Must be one of the options.                                                                                                                    |
| `options`           | **Required:** Array of all possible values.                                                                                                                                                                             |
| `entityId`          | The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. If not set, SmartBeans generates an entity ID using the bean name and field name. |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                                                                                            |
| `icon`              | The icon displayed in Home Assistant for this entity. If no icon is specified, it is derived from the device class.                                                                                                     |
| `deviceClass`       | _Not used for selects._                                                                                                                                                                                                 |
| `unitOfMeasurement` | _Not used for selects._                                                                                                                                                                                                 |
| `min`               | _Not used for selects._                                                                                                                                                                                                 |
| `max`               | _Not used for selects._                                                                                                                                                                                                 |
| `step`              | _Not used for selects._                                                                                                                                                                                                 |
| `displayMode`       | _Not used for selects._                                                                                                                                                                                                 |
| `pattern`           | _Not used for selects._                                                                                                                                                                                                 |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Query Current State

When the select entity’s state changes in Home Assistant, the update is immediately propagated to SmartBeans. You can 
query the current value at any time by calling the `getValue()` method on the `ConfigSelect` object.

## Create Entity Programmatically

In addition to the annotation-based approach, you can also create a configurable value programmatically using the 
`getConfigSelect()` method of the `SmartBeans` API. This approach is particularly useful when you want to use a type 
other than `String` for the selection. Keep in mind that in Home Assistant the type is always `String`; all type mapping
is handled internally by SmartBeans. Consequently, all possible values must be bidirectionally mappable to `String`
values.

### Using a Java Enum

Using a Java enum is straightforward, as the `SmartBeans` API provides a dedicated method for this use case. Simply call
the `getConfigSelect()` method and specify the enum you want to use. The resulting `ConfigSelect` automatically maps the
enum values to `String` using the standard `name()` and `valueOf()` methods of the enum class.  

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {

  public enum LightColor {
    white,
    red,
    yellow,
    pink
  }
  
  private ConfigSelect<LightColor> lightColor;

  @Entity("light.kitchen_ceiling")
  private Light ceiling;

  @Override
  public void init(SmartBeans sb) {
    lightColor = sb.getConfigSelect("lightColor", LightColor.white, LightColor.class);
  }
  
  public void onMotion() {
    ceiling.turnOn(LightAttr.colorName(lightColor.getValue().name()));
  }
}
````

### Using Any Type

Ultimately, you can use any type for `ConfigSelect` objects by defining how values are mapped to and from `String`. This
is done by providing a builder as the third argument.  

Consider the following example. While it may be somewhat contrived - since there are simpler ways to achieve the same
result - it illustrates the core idea in a concise way.  

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class KitchenMotionControl implements SmartBean {

  public record RGB(int red, int green, int blue) {

    public String print() {
      return red + "," + green + "," + blue;
    }

    public static RGB parse(String s) {
      String[] parts = s.split(",");
      return new RGB(
          Integer.parseInt(parts[0]), 
          Integer.parseInt(parts[1]), 
          Integer.parseInt(parts[2])
      );
    }
  }

  private ConfigSelect<RGB> lightColor;

  @Entity("light.kitchen_ceiling")
  private Light ceiling;

  @Override
  public void init(SmartBeans sb) {
    lightColor = sb.getConfigSelect("lightColor", new RGB(255, 0, 0), def -> def
        .setFromValueMapper(RGB::print)
        .setToValueMapper(RGB::parse)
        .setOptions(
            new RGB(255, 0, 0),
            new RGB(0, 255, 0),
            new RGB(0, 0, 255)
        )
        .setFriendlyName("Light color")
    );
  }

  public void onMotion() {
    ceiling.turnOn(LightAttr.rgbColor(
        lightColor.getValue().red(),
        lightColor.getValue().green(),
        lightColor.getValue().blue()
    ));
  }
}
````



