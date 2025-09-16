---
sidebar_position: 20
description: "Provided entity to to turn something on and off."
title: Switch
---

# Provided Switch

A provided switch can be used to turn something on/off and control a bean function with it. To create a provided switch,
declare a field of type `ProvidedSwitch` in your bean and annotate it with `@Provided`. SmartBeans will then automatically 
create the switch in Home Assistant, generate the corresponding Java object to handle state changes, and inject this
object into the annotated field. 

You can register listeners for the switch programmatically or use the `@OnSwitchTurnedOn` and `@OnSwitchTurnedOff`
annotations to mark a method. SmartBeans will then invoke the annotated method whenever the switch is turned on or off.

A provided switch can be used to toggle something on or off and link this action to a bean function. To create a
provided switch, declare a field of type `ProvidedSwitch` in your bean and annotate it with `@Provided`. SmartBeans 
will then automatically create the switch in Home Assistant, generate the corresponding Java object to handle 
state changes, and inject this object into the annotated field.  

You can either register listeners for the switch programmatically, or annotate methods with `@OnSwitchTurnedOn` and 
`@OnSwitchTurnedOff`. SmartBeans will then automatically invoke these methods whenever the switch is toggled on or off.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  @Provided(entityId = "switch.a_sample_switch")
  private ProvidedSwitch providedSwitch;

  @OnSwitchTurnedOn("providedSwitch")
  public void onSwitchTurnedOn() {
    //Switch turned on!
  }
}
````

:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no provided entities can be 
created, since they cannot be associated with a device.
:::

## Entity Configuration

The `@Provided` annotation can be used to configure the provided switch in detail. It supports the following 
attributes:

| Attribute           | Description                                                                                                                                         |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `entityId`          | **Required**. The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. |
| `id`                | The internal ID of the entity. This ID must be unique within the scope of the bean device. If not specified, the field name is used.                |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                        |
| `icon`              | The icon shown in Home Assistant for this entity. If no icon is specified, it is derived from the device class.                                     |
| `deviceClass`       | The [device class](https://www.home-assistant.io/integrations/switch/#device-class) of the entity in Home Assistant.                                |
| `unitOfMeasurement` | _Not used for buttons._                                                                                                                             |
| `stateClass`        | _Not used for buttons._                                                                                                                             |
| `options`           | _Not used for buttons._                                                                                                                             |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Handle Switch States

There are two main options for handling switch states: you can either annotate a bean method to be invoked when the switch 
is turned on or off, or register listeners on the `ProvidedSwitch` object.

### Annotation

You can annotate any method of your bean with the `@OnSwitchTurnedOn` or `@OnSwitchTurnedOff` annotation to have it 
invoked on state changes. Both annotations require the internal ID of the provided switch as its attribute, and the
annotated method must not have any parameters. If the `@Provided` annotation is used without specifying an internal ID,
the field name is used as the internal ID.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {
  
  private SmartBeans sb;

  @Provided(id = "my_id", entityId = "switch.a_sample_switch")
  private ProvidedSwitch providedSwitch;

  @OnSwitchTurnedOn("my_id")
  public void onTurnedOn() {
    sb.log("sample switch turned on");
  }

  @OnSwitchTurnedOff("my_id")
  public void onTurnedOff() {
    sb.log("sample switch turned off");
  }
}
````

### Listener

Alternatively, you can register listeners directly on the `ProvidedSwitch` instance. The listener must implement
the `ProvidedSwitch.TurnOnListener` and/or `ProvidedSwitch.TurnOffListener` interface, which each define a single
method: `onTurnedOn(TurnOnEvent event)` or `onTurnedOff(TurnOffEvent event)`. Since both are functional interfaces, they
can be implemented conveniently using lambda expressions.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  @Provided(entityId = "switch.a_sample_switch")
  private ProvidedSwitch providedSwitch;

  @Override
  public void init(SmartBeans sb) {
    providedSwitch.onTurnOn(evt -> sb.log("sample switch turned on"));
    providedSwitch.onTurnOff(evt -> sb.log("sample switch turned off"));
  }
}
````

### Setting the State

You can set the state of the switch from your bean by calling the `setState()` method on the `ProvidedSwitch`.  
When using this method, the state of the entity in Home Assistant will be updated if needed, but **no** `turnedOn` or 
`turnedOff` event will be fired in your bean.  

If, on the other hand, Home Assistant fires such an event, SmartBeans will automatically update the entity state 
accordingly — so you don’t need to manage synchronization yourself.  

Keep in mind that the state of the switch may need to be set explicitly when your bean is deployed. Otherwise, the 
previous state is retained across redeployments, which can either be intentional or undesired depending on your use case.

````java
public class ASampleBean implements SmartBean {

  @Provided(entityId = "switch.a_sample_switch")
  private ProvidedSwitch aSwitch;

  @Override
  public void init(SmartBeans sb) {
    aSwitch.setState(Switch.State.ON);
  }
}
````

## Create Entity Programmatically

In addition to the annotation-based approach, you can programmatically create a switch using the 
`provideSwitch()` method of the `SmartBeans` API. This approach is useful when the entity's attributes are 
generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the internal ID, which must be unique within the scope of the bean; 
the second is the entity ID used in Home Assistant; and the third is an optional builder for setting all other 
attributes.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {
  @Override
  public void init(SmartBeans sb) {
    ProvidedSwitch providedButton = sb.provideSwitch("my_switch", "switch.a_sample_switch", def -> def
        .setFriendlyName("A sample switch")
        .setIcon("mdi:home")
    );
    providedSwitch.onTurnOn(evt -> sb.log("sample switch turned on"));
  }
}
````
