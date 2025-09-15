---
sidebar_position: 20
description: "Entities of domain: light"
---

# Light

The `Light` interface represents light entities in Home Assistant. When you define a field of type `Light` in your
SmartBean class and annotate it with `@Entity`, the framework automatically injects an object that lets you control the
corresponding light entity in Home Assistant. This provides a type-safe way to interact with your smart lights.

````java
@Entity("light.kitchen_ceiling")
private Light light;
````

## State

The light's state is represented by the `Light.State` enum, which mirrors the possible states in Home Assistant: `ON`,
`OFF`, `UNKNOWN` and `UNAVAILABLE`. You can query the light's current state using these methods:

| Method               | Description                                                |
|----------------------|------------------------------------------------------------|
| `getState()`         | Get the current state of the light.                        |
| `isOn()`             | Shortcut to check if the light is currently switched on.   |
| `getStateAsString()` | Returns the original state from the Home Assistant entity. |

## Attributes

The following attributes of a light entity can be accessed through simple getter methods:

| HA attribute            | Method                     | Description                                                                 |
|-------------------------|----------------------------|-----------------------------------------------------------------------------|
| `friendly_name`         | `getFriendlyName()`        | Friendly name of the entity.                                                |
| `icon`                  | `getIcon()`                | Icon of the entity.                                                         |
| `brightness`            | `getBrightness()`          | Current brightness, `int` value between 0 and 255.                          |
| `supported_color_modes` | `getSupportedColorModes()` | All supported color modes by this light, list of `Light.ColorMode` enum.    |
| `color_mode`            | `getColorMode()`           | Current used color mode.                                                    |
| `color_temp_kelvin`     | `getColorTemp()`           | Current color temperature in kelvin, `int` value.                           |
| `min_color_temp_kelvin` | `getMinColorTemp()`        | Minimum supported color temperature in kelvin, `int` value.                 |
| `max_color_temp_kelvin` | `getMaxColorTemp()`        | Maximum supported color temperature in kelvin, `int` value.                 |
| `hs_color`              | `getHsColor()`             | Current light color in (h)ue and (s)aturation.                              |
| `rgb_color`             | `getRgbColor()`            | Current light color in (r)ed, (g)reen and (b)lue.                           |
| `rgbw_color`            | `getRgbwColor()`           | Current light color in (r)ed, (g)reen, (b)lue and (w)hite.                  |
| `rgbww_color`           | `getRgbwwColor()`          | Current light color in (r)ed, (g)reen, (b)lue, cold(w)hite and warm(w)hite. |
| `xy_color`              | `getXyColor()`             | Current light color in xy color.                                            |
| `effect_list`           | `getEffectList()`          | List of supported effects, list of `String`.                                |

You can access any additional attributes that are not directly supported through the `getAttributes()` method.

## Services

The `Light` interface provides several methods to control your light through Home Assistant services. Here are the
supported operations:

### `turnOn()`

Turns on the light with customizable parameters. This method accepts an array of `LightAttr` objects that specify how
the light should be turned on. These parameters correspond directly to Home Assistant's `light.turn_on` service
parameters. The `LightAttr` class provides convenient factory methods to create these parameters with proper typing and
validation.

````java
public class ASampleBean implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light light;

  public void someBeanMethod() {
    light.turnOn(LightAttr.brightnessPercent(30), LightAttr.transition(2));
  }
}
````

Using static imports can improve code readability and maintainability:

````java
import static io.github.beanssmart.ha.entities.LightAttr.*;

public class ASampleBean implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light light;

  public void someBeanMethod() {
    light.turnOn(brightnessPercent(30), transition(2));
  }
}
````

See the `LightAttr` class for convenient methods to create lighting parameters. These methods help write cleaner and 
more readable code, as shown in this example:

````java
public class ASampleBean implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light light;

  public void someBeanMethod() {
    light.turnOn(colorName("pink"), brightness(127), transition(ofSeconds(10)));
  }
}
````

### `turnOff()`

Turns off the light. While this method accepts `LightAttr` parameters (primarily for transition effects), note that
most light attributes don't apply to the off state.

````java
public class ASampleBean implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light light;

  public void someBeanMethod() {
    light.turnOff(LightAttr.transition(Duration.ofMillis(300)));
  }
}
````

### `toggle()`

To toggle a light on or off, use the `toggle()` method. It accepts the same parameters as the `turnOn()` method:

````java
public class ASampleBean implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light light;

  public void someBeanMethod() {
    light.toggle(LightAttr.rgbColor(255, 127, 0));
  }
}
````

## Access Entities Programmatically

In addition to the annotation-based approach, you can programmatically access lights using the `getLight()` 
method of the `SmartBeans` API. You might prefer this programmatic approach over annotations for example when the entity
ID is dynamically generated through business logic and cannot be determined at compile time.

````java
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  public void someBeanMethod() {
    Light light = sb.getLight("light.kitchen_ceiling");
    light.turnOn(LightAttr.brightness(150), LightAttr.colorName("white"));
  }
}
````

import UncachedNote from './_note_uncached_entities.md';

<UncachedNote />