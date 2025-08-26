---
sidebar_position: 20
description: "Entities of domain: vacuum"
---

# Vacuum

The `Vacuum` interface represents vacuum entities in Home Assistant. When you define a field of type `Vacuum` in your
SmartBean class and annotate it with `@Entity`, the framework automatically injects an object that lets you control the
corresponding vacuum entity in Home Assistant. This provides a type-safe way to interact with your vacuum robots.

````java
@Entity("vacuum.basement")
private Vacuum vacuum;
````

## State

The vacuums's state is represented by the `Vacuum.State` enum, which mirrors the possible 
[states](https://www.home-assistant.io/integrations/vacuum/#the-state-of-a-vacuum-entity) in Home Assistant: `CLEANING`, 
`DOCKED`, `IDLE`, `PAUSED`, `RETURNING`, `ERROR`, `UNKNOWN` and `UNAVAILABLE`. You can query the vacuums's current state
using these methods:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getState()`         | Get the current state of the vacuum.                       |
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a vacuum entity can be accessed through simple getter methods:

| HA attribute       | Method              | Description                                           |
|--------------------|---------------------|-------------------------------------------------------|
| `friendly_name`    | `getFriendlyName()` | Friendly name of the entity.                          |
| `icon`             | `getIcon()`         | Icon of the entity.                                   |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Services

The `Vacuum` interface provides several methods to control your vacuum through Home Assistant services. Here are the
supported operations:

| HA service              | Method           | Description                                     |
|-------------------------|------------------|-------------------------------------------------|
| `vacuum.start`          | `start()`        | Starts cleaning.                                |
| `vacuum.pause`          | `pause()`        | Pause cleaning.                                 |
| `vacuum.stop`           | `stop()`         | Stops cleaning.                                 |
| `vacuum.return_to_base` | `returnToBase()` | Returns the robot to it's base.                 |
| `vacuum.send_command`   | `sendCommand()`  | Send any vendor specific command to the vacuum. |

## Example

````java
public class ASampleBean implements SmartBean {

  @Entity("vacuum.basement")
  private Vacuum vacuum;

  public void someBeanMethod() {
    vacuum.sendCommand("clean_area", 
        atts -> atts.setAttribute("area_id", "1"));
  }
}
````

### Vendor specific services

Some vacuums provide vendor specific services, for example the roborock integration has a service `roborock.vacuum_clean_segment`.
These services don't have any coressponding methods, but you can call any service through the `SmartBeans` API, like this:

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  @Entity("vacuum.basement")
  private Vacuum vacuum;

  public void someBeanMethod() {
    sb.callService(
        Service.name("roborock.vacuum_clean_segment")
            .onTarget(Target.entity(vacuum))
            .withData(atts -> atts.setAttributeList("segments", 1, 2, 3))
    );
  }
}
````

## Access entities programmatically

In addition to the annotation-based approach, you can programmatically access vacuum entities using the `getVacuum()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    Vacuum robot = sb.getVacuum("vacuum.robot");
    if(robot.getState() == Vacuum.State.CLEANING) {
      robot.returnToBase();
    }
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />