---
sidebar_position: 20
description: "Entities of domain: camera"
---

# Camera

The `Camera` interface represents camera entities in Home Assistant. When you define a field of type `Camera` in your
SmartBean class and annotate it with `@Entity`, the framework automatically injects an object that lets you control the
corresponding camera entity in Home Assistant. This provides a type-safe way to interact with your cameras.

````java
@Entity("camera.stairway")
private Camera camera;
````

## State

The camera's state is represented by the `Camera.State` enum, which mirrors the possible states in Home Assistant: 
`RECORDING`, `STREAMING`, `IDLE`, `UNKNOWN` and `UNAVAILABLE`. You can query the camera's current state using these methods:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getState()`         | Get the current state of the camera.                       |
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a camera entity can be accessed through simple getter methods:

| HA attribute       | Method              | Description                                           |
|--------------------|---------------------|-------------------------------------------------------|
| `friendly_name`    | `getFriendlyName()` | Friendly name of the entity.                          |
| `icon`             | `getIcon()`         | Icon of the entity.                                   |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Services

The `Camera` interface provides several methods to control your camera through Home Assistant services. Here are the
supported operations:

### `turnOn()`, `turnOff()`

Turns the camera on or off.

````java
public class ASampleBean implements SmartBean {

  @Entity("camera.stairway")
  private Camera camera;

  public void someBeanMethod() {
    camera.turnOn();
  }
}
````

### `playStream()`

The method `playStream()` can be used to stream the picture of the camera to a [media player](./media-player) entity, represented by a
`MediaPlayer` object.

````java
public class ASampleBean implements SmartBean {

  @Entity("camera.stairway")
  private Camera camera;
  
  @Entity("media_player.tv")
  private MediaPlayer tv;

  public void someBeanMethod() {
    camera.playStream(tv, Camera.StreamType.WEB_RTC);
  }
}
````

### `record()`

Records the current stream of the camera to a file. Additionally this method accepts an array of `CameraAttr` objects 
that specify the video is recorded. These parameters correspond directly to Home Assistant's `camera.record`
service parameters. The `CameraAttr` class provides convenient factory methods to create these parameters with proper
typing and validation.

````java
public class ASampleBean implements SmartBean {

  @Entity("camera.stairway")
  private Camera camera;

  public void someBeanMethod() {
    camera.record("/tmp/stairway.mp4", CameraAttr.duration(Duration.ofMinutes(2)));
  }
}
````

### `snapshot()`

Take a snapshot of the camera and write it to a file.

````java
public class ASampleBean implements SmartBean {

  @Entity("camera.stairway")
  private Camera camera;

  public void someBeanMethod() {
    camera.snapshot("/tmp/stairway.jpg");
  }
}
````

## Access Entities Programmatically

In addition to the annotation-based approach, you can programmatically access cameras using the `getCamera()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    Camera camera = sb.getCamera("camera.frontdoor");
    camera.snapshot("/tmp/snapshot_frontdoor.jpg");
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />