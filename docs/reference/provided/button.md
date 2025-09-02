---
sidebar_position: 20
description: "Provided entity to trigger bean actions from Home Assistant"
title: Button
---

# Provided button

A provided button can be used to trigger bean actions from Home Assistant, either manually by the user or through 
automations. To create a provided button, declare a field of type `ProvidedButton` in your bean and annotate it with 
`@Provided`. SmartBeans will then automatically create the button in Home Assistant, generate the corresponding Java 
object to handle button presses, and inject this object into the annotated field. 

You can register listeners for the button programmatically or use the `@OnButtonPressed` annotation to mark a method. 
SmartBeans will then invoke the annotated method whenever the button is pressed.


````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  @Provided(entityId = "button.a_sample_button")
  private ProvidedButton providedButton;

  @OnButtonPressed("providedButton")
  public void onSampleButtonPressed() {
    //Action triggered!
  }
}
````

:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no provided entities can be 
created, since they cannot be associated with a device.
:::

## Entity configuration

The `@Provided` annotation can be used to configure the provided binary sensor in detail. It supports the following 
attributes:

| Attribute           | Description                                                                                                                                         |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `entityId`          | **Required**. The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. |
| `id`                | The internal ID of the entity. This ID must be unique within the scope of the bean device. If not specified, the field name is used.                |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                        |
| `icon`              | The icon shown in Home Assistant for this entity. If no icon is specified, it is derived from the device class.                                     |
| `deviceClass`       | The [device class](https://www.home-assistant.io/integrations/button/#device-class) of the entity in Home Assistant.                                |
| `unitOfMeasurement` | _Not used for buttons._                                                                                                                             |
| `stateClass`        | _Not used for buttons._                                                                                                                             |
| `options`           | _Not used for buttons._                                                                                                                             |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Handle button presses

There are two main options for handling button presses: you can either annotate a bean method to be invoked on button 
presses, or register a listener on the `ProvidedButton` object.

### Annotation

You can annotate any method of your bean with the `@OnButtonPressed` annotation to have it invoked on button presses. 
The annotation requires the internal ID of the provided button as its attribute, and the annotated method must not have
any parameters. If the `@Provided` annotation is used without specifying an internal ID, the field name is used as the 
internal ID.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {
  
  private SmartBeans sb;

  @Provided(id = "my_id", entityId = "button.a_sample_button")
  private ProvidedButton providedButton;

  @OnButtonPressed("my_id")
  public void onButtonPressed() {
    sb.log("sample button pressed");
  }
}
````

### Listener

The alternative approach is to register a listener on the `ProvidedButton` instance. The listener must implement the 
`ProvidedButton.Listener` interface, which defines a single method, `onButtonPressed()`, taking a `PressedEvent` as its
parameter. This allows the listener to be implemented as a lambda expression.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  @Provided(entityId = "button.a_sample_button")
  private ProvidedButton providedButton;

  @Override
  public void init(SmartBeans sb) {
    providedButton.onPressed(evt -> sb.log("sample button pressed"));
  }
}
````

## Create entity programmatically

In addition to the annotation-based approach, you can programmatically create a button using the 
`provideButton()` method of the `SmartBeans` API. This approach is useful when the entity's attributes are 
generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the internal ID, which must be unique within the scope of the bean; 
the second is the entity ID used in Home Assistant; and the third is an optional builder for setting all other 
attributes.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {
  @Override
  public void init(SmartBeans sb) {
    ProvidedButton providedButton = sb.provideButton("my_button", "button.a_sample_button", def -> def
        .setFriendlyName("A sample button")
        .setIcon("mdi:home")
    );
    providedButton.onPressed(evt -> sb.log("sample button pressed"));
  }
}
````
