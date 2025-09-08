---
sidebar_position: 20
description: "A number representing a numeric config parameter."
title: Number
---

# Number config

The `ConfigNumber` can be used to create a configuration entity in Home Assistant for defining any number. Technically, 
it represents a configurable number value in the bean that can be controlled through a Home Assistant number entity. 
The value can be queried within your bean's code to influence its behavior. 

To create a configurable number, declare a field of type `ConfigNumber` in your bean and annotate it with `@Config`. 
The annotation's attributes allow you to configure the details of the corresponding entity in Home Assistant. The `value`
attribute specifies the default or initial value and must be set to a valid `integer`.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class HeatingControl implements SmartBean {
  
  @Config(value = "20")
  public ConfigNumber targetTemperature;
  
  @Entity("switch.bathroom_heater")
  public Switch heater;

  @OnNumericStateTrigger(entity = "sensor.bathroom_temperature")
  public void triggerRoutine(StateEvent evt) {
    if(evt.getNewState().getStateAsInteger() < targetTemperature.toInt()) {
      heater.turnOn();
    }
    else {
      heater.turnOff();
    }
  }
}
````

:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no config entities can be 
created, since they cannot be associated with a device.
:::

## Entity configuration

The `@Config` annotation can be used to configure the number entity created for the configurable duration in detail. It
supports the following attributes:

| Attribute           | Description                                                                                                                                                                                                             |
|---------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `value`             | **Required:** The default or initial value of the configurable duration. Must be a valid `integer`.                                                                                                                     |
| `entityId`          | The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. If not set, SmartBeans generates an entity ID using the bean name and field name. |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                                                                                            |
| `icon`              | The icon displayed in Home Assistant for this entity. If no icon is specified, it is derived from the device class.                                                                                                     |
| `deviceClass`       | The [device class](https://www.home-assistant.io/integrations/number/#device-class) of the number entity in Home Assistant.                                                                                             |
| `unitOfMeasurement` | The unit used, when displaying the number entity's value. If not specified, the unit is derived from the device class.                                                                                                  |
| `min`               | The minimum allowed value of the number entity. Default is `1`.                                                                                                                                                         |
| `max`               | The maximum allowed value of the number entity. Default is `100`.                                                                                                                                                       |
| `step`              | The increment step for adjusting the number entity's value. Default is `1`.                                                                                                                                             |
| `displayMode`       | The display mode of the number entity in Home Assistant. See the `mode` property of the [MQTT number entity](https://www.home-assistant.io/integrations/number.mqtt/) for possible values. Default is `auto`.           |
| `options`           | _Not used for numbers._                                                                                                                                                                                                 |
| `pattern`           | _Not used for numbers._                                                                                                                                                                                                 |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Query current state

When the number entity’s state changes in Home Assistant, the update is immediately propagated to SmartBeans. You can 
query the current value at any time by calling the appropriate methods on the `ConfigNumber` object. The value can be 
retrieved either as an `integer` or as a `double`.

| Method       | Description                                |
|--------------|--------------------------------------------|
| `toInt()`    | Returns the current value as an `integer`. |
| `toDouble()` | Returns the current value as a `double`.   |

Here is a complete example:

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ClimateControl implements SmartBean {

  @Config(
      value = "20",
      min = 15,
      max = 25,
      step = 0.5,
      unitOfMeasurement = "°C",
      displayMode = "slider",
      friendlyName = "Target temperature bathroom"
  )
  public ConfigNumber targetTemperature;

  @Entity("climate.bathroom")
  public Climate heater;

  @OnNumericStateTrigger(entity = "sensor.bathroom_temperature")
  public void triggerRoutine(StateEvent evt) {
    if(evt.getNewState().getStateAsDouble() < targetTemperature.toDouble()) {
      heater.setHvacMode(Climate.HvacMode.HEAT);
    }
    else if(evt.getNewState().getStateAsDouble() > targetTemperature.toDouble()) {
      heater.setHvacMode(Climate.HvacMode.COOL);
    }
  }
}
````

## Create entity programmatically

In addition to the annotation-based approach, you can programmatically create a configurable number using the 
`getConfigNumber()` method of the `SmartBeans` API. This approach is useful in some edge cases when the entity's
attributes are generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the configuration name, which must be unique within the scope of the
bean; the second is the default or initial value; and the third is a builder for defining all other attributes of the 
entity in Home Assistant.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ClimateControl implements SmartBean {

  public ConfigNumber targetTemperature;

  @Override
  public void init(SmartBeans sb) {
    targetTemperature = sb.getConfigNumber("targetTemp", 20, def -> def
        .setMin(15).setMax(25).setStep(0.5)
        .setDisplayMode(DisplayMode.SLIDER)
        .setUnitOfMeasurement("°C")
        .setFriendlyName("Target temperature bathroom")
    );
  }
}
````
