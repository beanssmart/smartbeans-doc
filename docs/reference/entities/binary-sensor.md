---
sidebar_position: 20
description: "Entities of domain: binary_sensor"
---

# Binary Sensor

The `BinarySensor` interface represents `binary_sensor` entities in Home Assistant. When you define a field of type 
`BinarySensor` in your SmartBean class and annotate it with `@Entity`, the framework automatically injects an object 
that lets you control the corresponding entity in Home Assistant.

````java
@Entity("binary_sensor.motion_kitchen")
private BinarySensor motion;
````

## State

The sensor's state is represented by the `BinarySensor.State` enum, which mirrors the possible states in Home Assistant: `ON`,
`OFF`, `UNKNOWN` and `UNAVAILABLE`. You can query the sensor's current state using these methods:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getState()`         | Get the current state of the sensor.                       |
| `isOn()`             | Shortcut to check if the sensor is currently on.           |
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a binary sensor entity can be accessed through simple getter methods:

| HA attribute    | Method               | Description                                                                                           |
|-----------------|----------------------|-------------------------------------------------------------------------------------------------------|
| `friendly_name` | `getFriendlyName()`  | Friendly name of the entity.                                                                          |
| `icon`          | `getIcon()`          | Icon of the entity.                                                                                   |
| `device_class`  | `getDeviceClass()`   | [Device class](https://www.home-assistant.io/integrations/binary_sensor/#device-class) of the sensor. |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Example

````java
public class ASampleBean implements SmartBean {
  
  private SmartBeans sb;

  @Entity("binary_sensor.occupancy_kitchen")
  private BinarySensor occupancy;

  public void someBeanMethod() {
    if(occupancy.isOn()) {
      sb.log("kitchen is occupied");
    }
    else {
      sb.log("kitchen is free");
    }
  }
}
````

## Access Entities Programmatically

In addition to the annotation-based approach, you can programmatically access binary sensors using the `getBinarySensor()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    BinarySensor binarySensor = sb.getBinarySensor("binary_sensor.movement_kitchen");
    if(binarySensor.isOn()) {
      sb.log("There is movement in the kitchen.");
    }
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />