---
sidebar_position: 20
description: "Entities of domain: scene"
---

# Scene

The `Scene` interface represents a scene in Home Assistant. When you define a field of type `Scene` in your
SmartBean class and annotate it with `@Entity`, the framework automatically injects an object that lets you control the
corresponding scene in Home Assistant.

````java
@Entity("scene.cosy_light")
private Scene scene;
````

## State

Scenes in Home Assistant maintain a state that represents the timestamp of their last activation. While this state might 
not be directly useful for automation logic, you can query it if needed using this method:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a scene can be accessed through simple getter methods:

| HA attribute       | Method              | Description                                           |
|--------------------|---------------------|-------------------------------------------------------|
| `friendly_name`    | `getFriendlyName()` | Friendly name of the entity.                          |
| `icon`             | `getIcon()`         | Icon of the entity.                                   |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Activate Scene

The `Scene` interface provides a `turnOn()` method to activate scenes. This method accepts an optional `double` 
parameter that specifies the transition duration in seconds. When no transition time is provided, the scene activates 
immediately.

````java
public class ASampleBean implements SmartBean {

  @Entity("scene.cosy_light")
  private Scene scene;

  public void someBeanMethod() {
    scene.turnOn(1.5);
  }
}
````

## Access entities programmatically

In addition to the annotation-based approach, you can programmatically access scenes using the `getScene()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    Scene scene = sb.getScene("scene.cosy_light");
    scene.turnOn(5);
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />