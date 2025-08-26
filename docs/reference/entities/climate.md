---
sidebar_position: 20
description: "Entities of domain: climate"
---

# Climate

The `Climate` interface represents climate entities in Home Assistant. When you define a field of type `Climate` in your
SmartBean class and annotate it with `@Entity`, the framework automatically injects an object that lets you control the
corresponding climate entity in Home Assistant.

````java
@Entity("climate.hvac_livingroom")
private Climate climate;
````

## State

The climate's state is represented by the `Climate.State` enum, which mirrors the possible states in Home Assistant: `OFF`,
`HEAT`, `COOL`, `HEAT_COOL`, `AUTO`, `DRY`, `FAN_ONLY`, `UNKNOWN` and `UNAVAILABLE`. You can query the climate's current
state using these methods:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getState()`         | Get the current state of the climate.                      |
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a climate entity can be accessed through simple getter methods:

| HA attribute          | Method                    | Description                                                                                                                   |
|-----------------------|---------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| `friendly_name`       | `getFriendlyName()`       | Friendly name of the entity.                                                                                                  |
| `icon`                | `getIcon()`               | Icon of the entity.                                                                                                           |
| `current_temperature` | `getCurrentTemperature()` | Current temperature, measured by the device.                                                                                  |
| `current_humidity`    | `getCurrentHumidity()`    | Current humidity, measured by the device.                                                                                     |
| `fan_mode`            | `getFanMode()`            | Current mode of the fan, if present.                                                                                          |
| `fan_modes`           | `getFanModes()`           | All possible values of `fan_mode` attribute.                                                                                  |
| `hvac_mode`           | `getHvacMode()`           | Current HVAC mode of the device.                                                                                              |
| `hvac_modes`          | `getHvacModes()`          | All HVAC modes supported by this device. Possible values are `OFF`, `HEAT`, `COOL`, `HEAT_COOL`, `AUTO`, `DRY` and `FAN_ONLY` |
| `temperature`         | `getTemperature()`        | Current target temperature.                                                                                                   |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Services

The `Climate` interface provides several methods to control your climate through Home Assistant services. Here are the
supported operations:

### `setTemperature()`

Sets the target temperatur of the device. This method accepts an array of `ClimateAttr` objects that specify how
the target temperature is defined. These parameters correspond directly to Home Assistant's `climate.set_temperature`
service parameters. The `ClimateAttr` class provides convenient factory methods to create these parameters with proper
typing and validation.

````java
public class ASampleBean implements SmartBean {

  @Entity("climate.hvac_livingroom")
  private Climate climate;

  public void someBeanMethod() {
    climate.setTemperature(ClimateAttr.temperature(21));
  }
}
````

Using static imports can improve code readability and maintainability:

````java
public class ASampleBean implements SmartBean {

  @Entity("climate.hvac_livingroom")
  private Climate climate;

  public void someBeanMethod() {
    climate.setTemperature(
        targetTemperatureLow(20), 
        targetTemperatureHigh(23), 
        hvacMode(HvacMode.AUTO));
  }
}
````

### `setHvacMode()`

Sets the operating mode of the climate device. Takes the mode to be set as an parameter.

````java
public class ASampleBean implements SmartBean {

  @Entity("climate.hvac_livingroom")
  private Climate climate;

  public void someBeanMethod() {
    climate.setHvacMode(HvacMode.FAN_ONLY);
  }
}
````

### `turnOn()`, `turnOff()`

Turns on or off the climate device.

````java
public class ASampleBean implements SmartBean {

  @Entity("climate.hvac_livingroom")
  private Climate climate;

  public void someBeanMethod() {
    climate.turnOn();
  }
}
````

### Other services

There are other services in the climate domain, which are currently not supported via the `Climate` interface, but can be
called through the `SmartBeans` API, if needed. For example `climate.set_preset_mode` sets a preset mode like 
`vacation` to the climate device.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  @Entity("climate.hvac_livingroom")
  private Climate climate;

  public void someBeanMethod() {
    sb.callService(Service.name("climate.set_preset_mode")
        .onTarget(Target.entity(climate))
        .withData(atts -> atts.setAttribute("preset_mode", "vacation"))
    );
  }
}
````

## Access entities programmatically

In addition to the annotation-based approach, you can programmatically access climate entities using the `getClimate()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    Climate climate = sb.getClimate("climate.bathroom_heating");
    climate.setTemperature(ClimateAttr.temperature(23));
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />