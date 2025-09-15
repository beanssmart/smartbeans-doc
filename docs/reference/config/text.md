---
sidebar_position: 20
description: "A text representing a string config parameter."
title: Text
---

# Text Config

The `ConfigText` can be used to create a configuration entity in Home Assistant for defining any String. Technically, 
it represents a configurable String value in the bean that can be controlled through a Home Assistant text entity. 
The value can be queried within your bean's code to influence its behavior. 

To create a configurable String, declare a field of type `ConfigText` in your bean and annotate it with `@Config`. 
The annotation's attributes allow you to configure the details of the corresponding entity in Home Assistant. The `value`
attribute specifies the default or initial value.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class GreetingBean implements SmartBean {

  private SmartBeans sb;

  @Config(value = "Hello!")
  public ConfigText greetingPhrase;

  public void greet() {
    sb.callService(Service.name("notify.notify")
        .withData(data -> data.setAttribute("message", greetingPhrase.getValue()))
    );
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

| Attribute           | Description                                                                                                                                                                                                             |
|---------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `value`             | **Required:** The default or initial value of the configurable value.                                                                                                                                                   |
| `entityId`          | The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. If not set, SmartBeans generates an entity ID using the bean name and field name. |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                                                                                            |
| `icon`              | The icon displayed in Home Assistant for this entity. If no icon is specified, it is derived from the device class.                                                                                                     |
| `displayMode`       | Display mode of the text field in Home Assistant, can be `text` or `password`. Default is `text`.                                                                                                                       |
| `deviceClass`       | _Not used for strings._                                                                                                                                                                                                 |
| `unitOfMeasurement` | _Not used for strings._                                                                                                                                                                                                 |
| `min`               | _Not used for strings._                                                                                                                                                                                                 |
| `max`               | _Not used for strings._                                                                                                                                                                                                 |
| `step`              | _Not used for strings._                                                                                                                                                                                                 |
| `options`           | _Not used for strings._                                                                                                                                                                                                 |
| `pattern`           | _Not used for strings._                                                                                                                                                                                                 |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Query Current State

When the text entity’s state changes in Home Assistant, the update is immediately propagated to SmartBeans. You can 
query the current value at any time by calling the `getValue()` method on the `ConfigText` object as shown in the above 
example.

## Create Entity Programmatically

In addition to the annotation-based approach, you can programmatically create a configurable String value using the 
`getConfigText()` method of the `SmartBeans` API. This approach is useful in some edge cases when the entity's
attributes are generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the configuration name, which must be unique within the scope of the
bean; the second is the default or initial value; and the third is a builder for defining all other attributes of the 
entity in Home Assistant.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class GreetingBean implements SmartBean {

  public ConfigText greetingPhrase;

  @Override
  public void init(SmartBeans sb) {
    greetingPhrase = sb.getConfigText("greeting", "Hello!", def -> def
        .setFriendlyName("Greeting phrase")
    );
  }
}
````
