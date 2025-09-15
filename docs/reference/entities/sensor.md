---
sidebar_position: 20
description: "Entities of domain: sensor"
---

# Sensor

The `Sensor` interface represents `sensor` entities in Home Assistant. When you define a field of type 
`Sensor` in your SmartBean class and annotate it with `@Entity`, the framework automatically injects an object 
that lets you control the corresponding entity in Home Assistant.

````java
@Entity("sensor.temperature_bathroom")
private Sensor temperature;
````

## State

The sensor's state in Home Assistant is represented by a `String`. The Sensor interface provides convenience methods to
retrieve the state in commonly used datatypes for easier handling. You can query the sensor's current state using these 
methods:

| Method               | Description                                                                                                                                         |
|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `getState()`         | Get the current state of the sensor as String including unit of measurement.                                                                        |
| `getStateAsString()` | Returns the original state from the Home Assistant entity.                                                                                          |
| `asInteger()`        | Returns the state as `Integer`, returns `null`, if current state cannot be parsed to an integer, for example if it's `unavailable`                  |
| `asInt(int)`         | Returns the state as `int`, returns the provided default value, if current state cannot be parsed to an integer, for example if it's `unavailable`  |
| `asDouble()`         | Returns the state as `Double`, returns `null`, if current state cannot be parsed to a double, for example if it's `unavailable`                     |
| `asDouble(double)`   | Returns the state as `double`, returns the provided default value, if current state cannot be parsed to a double, for example if it's `unavailable` |

## Attributes

The following attributes of a sensor entity can be accessed through simple getter methods:

| HA attribute          | Method                   | Description                                                                                                         |
|-----------------------|--------------------------|---------------------------------------------------------------------------------------------------------------------|
| `friendly_name`       | `getFriendlyName()`      | Friendly name of the entity.                                                                                        |
| `icon`                | `getIcon()`              | Icon of the entity.                                                                                                 |
| `unit_of_measurement` | `getUnitOfMeasurement()` | Unit of measurement of the sensors's value.                                                                         |
| `device_class`        | `getDeviceClass()`       | [Device class](https://www.home-assistant.io/integrations/sensor/#device-class) of the sensor.                      |
| `state_class`         | `getStateClass()`        | [State class](https://developers.home-assistant.io/docs/core/entity/sensor/#available-state-classes) of the sensor. |
| `options`             | `getOptions()`           | List of available options of the sensor, provided if sensor has a enum-like state.                                  |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Example

````java
public class ASampleBean implements SmartBean {
  
  private SmartBeans sb;

  @Entity("sensor.bathroom_temperature")
  private Sensor temperature;

  public void someBeanMethod() {
    if(temperature.asInt(0) < 15) {
      sb.log("Bathroom is too cold! It's only " + temperature.getState());
    }
  }
}
````

## Access Entities Programmatically

In addition to the annotation-based approach, you can programmatically access sensors using the `getSensor()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    Sensor sensor = sb.getSensor("sensor.temperature_outside");
    if(sensor.asDouble(0d) > 20) {
      sb.log("Outside temperature is above 20" + sensor.getUnitOfMeasurement());
    }
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />