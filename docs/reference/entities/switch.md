---
sidebar_position: 20
description: "Entities of domain: switch"
---

# Switch

The `Switch` interface represents switch entities in Home Assistant. When you define a field of type `Switch` in your
SmartBean class and annotate it with `@Entity`, the framework automatically injects an object that lets you control the
corresponding switch entity in Home Assistant. This provides a type-safe way to interact with your smart switches.

````java
@Entity("switch.outlet")
private Switch outlet;
````

## State

The switch's state is represented by the `Switch.State` enum, which mirrors the possible states in Home Assistant: `ON`,
`OFF`, `UNKNOWN` and `UNAVAILABLE`. You can query the switch's current state using these methods:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getState()`         | Get the current state of the switch.                       |
| `isOn()`             | Shortcut to check if the switch is currently switched on.  |
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a switch entity can be accessed through simple getter methods:

| HA attribute       | Method              | Description                                           |
|--------------------|---------------------|-------------------------------------------------------|
| `friendly_name`    | `getFriendlyName()` | Friendly name of the entity.                          |
| `icon`             | `getIcon()`         | Icon of the entity.                                   |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Services

The `Switch` interface provides several methods to control your switch through Home Assistant services. Here are the
supported operations:

| HA service        | Method      | Description           |
|-------------------|-------------|-----------------------|
| `switch.turn_on`  | `turnOn()`  | Turns on the switch.  |
| `switch.turn_off` | `turnOff()` | Turns off the switch. |
| `switch.toggle`   | `toggle()`  | Toggles the switch.   |

## Example

````java
public class ASampleBean implements SmartBean {

  @Entity("switch.outlet")
  private Switch outlet;

  public void someBeanMethod() {
    outlet.toggle();
  }
}
````

## Access Entities Programmatically

In addition to the annotation-based approach, you can programmatically access switch entities using the `getSwitch()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    Switch outlet = sb.getSwitch("switch.outlet");
    if(outlet.isOn()) {
      outlet.turnOff();
    }
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />