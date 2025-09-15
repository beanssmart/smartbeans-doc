---
sidebar_position: 20
description: "Entities of domain: cover"
---

# Cover

The `Cover` interface represents cover entities in Home Assistant. When you define a field of type `Cover` in your
SmartBean class and annotate it with `@Entity`, the framework automatically injects an object that lets you control the
corresponding cover entity in Home Assistant. This provides a type-safe way to interact with your smart covers.

````java
@Entity("cover.kitchen_window")
private Cover cover;
````

## State

The cover's state is represented by the `Cover.State` enum, which mirrors the possible states in Home Assistant: `CLOSED`,
`CLOSING`, `OPEN`, `OPENING`, `UNKNOWN` and `UNAVAILABLE`. You can query the cover's current state using these methods:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getState()`         | Get the current state of the cover.                        |
| `isOpen()`           | Shortcut to check if the cover is currently opened.        |
| `isClosed()`         | Shortcut to check if the cover is currently closed.        |
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a cover entity can be accessed through simple getter methods:

| HA attribute       | Method              | Description                                           |
|--------------------|---------------------|-------------------------------------------------------|
| `friendly_name`    | `getFriendlyName()` | Friendly name of the entity.                          |
| `icon`             | `getIcon()`         | Icon of the entity.                                   |
| `current_position` | `getPosition()`     | Current position, `int` value between 0 and 100.      |
| `tilt_position`    | `getTiltPosition()` | Current tilt position, `int` value between 0 and 100. |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Services

The `Cover` interface provides several methods to control your cover through Home Assistant services. Here are the
supported operations:

| HA service                      | Method                 | Description                                                 |
|---------------------------------|------------------------|-------------------------------------------------------------|
| `cover.open_cover`              | `open()`               | Opens the cover.                                            |
| `cover.close_cover`             | `close()`              | Closes the cover.                                           |
| `cover.stop_cover`              | `stop()`               | Stops the cover.                                            |
| `cover.toggle`                  | `toggle()`             | Toggles the cover.                                          |
| `cover.open_cover_tilt`         | `openTilt()`           | Opens the cover tilt.                                       |
| `cover.close_cover_tilt`        | `closeTilt()`          | Closes the cover tilt.                                      |
| `cover.stop_cover_tilt`         | `stopTilt()`           | Stops the cover tilt.                                       |
| `cover.toggle_tilt`             | `toggleTilt()`         | Toggle the cover tilt.                                      |
| `cover.set_cover_position`      | `setPosition(int)`     | Sets the position of the cover to the specified value.      |
| `cover.set_cover_tilt_position` | `setTiltPosition(int)` | Sets the position of the cover tilt to the specified value. |

## Example

````java
public class ASampleBean implements SmartBean {

  @Entity("cover.kitchen_window")
  private Cover cover;

  public void someBeanMethod() {
    cover.setPosition(30);
  }
}
````

## Access Entities Programmatically

In addition to the annotation-based approach, you can programmatically access cover entities using the `getCover()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    Cover cover = sb.getCover("cover.office");
    cover.close();
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />