---
sidebar_position: 20
description: "Provided entity that looks and behaves like a light in Home Assistant."
title: Light
---

# Provided Light

A provided light can be used to toggle something on or off, control brightness and/or color and link these actions to 
bean functions. To create a provided light, declare a field of type `ProvidedLight` in your bean and annotate it with 
`@Provided`. SmartBeans will then automatically create the light in Home Assistant, generate the corresponding Java 
object to handle state changes, and inject this object into the annotated field.  

You can either register listeners for the light programmatically, or annotate methods with `@OnLightTurnedOn`, 
`@OnLightTurnedOff`, `@OnLightBrightnessChanged` and `@OnLightColorChanged`. SmartBeans will then automatically invoke 
these methods whenever the light changes the corresponding state.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  private SmartBeans sb;

  @Provided(entityId = "switch.a_sample_light", supportBrightness = true, colorModes = Light.ColorMode.RGB)
  private ProvidedLight providedLight;

  @OnLightTurnedOn("providedLight")
  public void onLightTurnedOn() {
    sb.log("Light switched on.");
  }

  @OnLightColorChanged("providedLight")
  public void onLightColorChange(ProvidedLight.ColorEvent event) {
    sb.log("Light color changed to " + event.getRgbColor() + ".");
  }
}
````

:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no provided entities can be 
created, since they cannot be associated with a device.
:::

## Entity Configuration

The `@Provided` annotation can be used to configure the provided light in detail. It supports the following 
attributes:

| Attribute           | Description                                                                                                                                         |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `entityId`          | **Required**. The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. |
| `id`                | The internal ID of the entity. This ID must be unique within the scope of the bean device. If not specified, the field name is used.                |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                        |
| `icon`              | The icon shown in Home Assistant for this entity. If no icon is specified, it is derived from the device class.                                     |
| `supportBrightness` | Whether the light supports brightness control. Defaults to `true`.                                                                                  |
| `colorModes`        | A list of all color modes supported by the light.                                                                                                   |
| `deviceClass`       | _Not used for lights._                                                                                                                              |
| `unitOfMeasurement` | _Not used for lights._                                                                                                                              |
| `stateClass`        | _Not used for lights._                                                                                                                              |
| `options`           | _Not used for lights._                                                                                                                              |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Handle Light States

There are two main options for handling light states: you can either annotate a bean method to be invoked when the light
is turned on or off, or when its brightness or color changes; or you can register listeners directly on the `ProvidedLight`
object to react to these state changes.

### Annotation

You can annotate any method of your bean with the `@OnLightTurnedOn`, `@OnLightTurnedOff`, `@OnLightBrightnessChanged`,  
or `@OnLightColorChanged` annotation to have it invoked on corresponding state changes. All annotations require the 
internal ID of the provided light as their attribute. The annotated method can either have no parameters or accept a
single parameter of the corresponding event type. If the `@Provided` annotation is used without specifying an internal 
ID, the field name of the light is used as the internal ID.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {
  
  private SmartBeans sb;

  @Provided(id = "my_id", entityId = "light.a_sample_light", colorModes = Light.ColorMode.HS)
  private ProvidedLight providedLight;

  @OnLightTurnedOn("my_id")
  public void onTurnedOn() {
    sb.log("sample light turned on");
  }

  @OnLightTurnedOff("my_id")
  public void onTurnedOff() {
    sb.log("sample light turned off");
  }

  @OnLightBrightnessChanged("my_id")
  public void onBrightnessChanged(ProvidedLight.BrightnessEvent evt) {
    sb.log("sample light has now brightness " + evt.getBrightnessPercent() + "%");
  }
  
  @OnLightColorChanged("my_id")
  public void onColorChanged(ProvidedLight.ColorEvent evt) {
    sb.log("sample light has now color hue " + evt.getHsColor().hue() + " with saturation " + evt.getHsColor().saturation());
  }
}
````

### Listener

Alternatively, you can register listeners directly on the `ProvidedLight` instance. The listener must implement
the `ProvidedLight.Listener` interface, which defines methods for every state chage: `onTurnedOn(TurnOnEvent event)`, 
`onTurnedOff(TurnOffEvent event)`, `onBrightnessChanged(BrightnessEvent event)` and `onColorChanged(ColorEvent event)`.


````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  @Provided(entityId = "light.a_sample_light", supportBrightness = true, colorModes = Light.ColorMode.XY)
  private ProvidedLight providedLight;

  @Override
  public void init(SmartBeans sb) {
    providedLight.addListener(new ProvidedLight.Listener() {
      @Override
      public void onTurnedOn(ProvidedLight.TurnOnEvent event) {
        sb.log("sample light turned on");
      }

      @Override
      public void onTurnedOff(ProvidedLight.TurnOffEvent event) {
        sb.log("sample light turned off");
      }

      @Override
      public void onBrightnessChanged(ProvidedLight.BrightnessEvent event) {
        sb.log("sample light changed brightness to " + event.getBrightness());
      }

      @Override
      public void onColorChanged(ProvidedLight.ColorEvent event) {
        sb.log("sample light changed color to " + event.getXyColor());
      }
    });
  }
}
````

### Setting the State

You can set the state of the light from your bean by calling the `setState()` method on the `ProvidedLight`.  
When using this method, the state of the entity in Home Assistant will be updated if needed, but **no** `turnedOn` or 
`turnedOff` event will be fired in your bean. The same applies to the `setBrightness()` and `setColor()` methods.

If, on the other hand, Home Assistant fires such an event, SmartBeans will automatically update the entity state 
accordingly — so you don’t need to handle synchronization manually.  

Keep in mind that the state of the light may need to be set explicitly when your bean is deployed. Otherwise, the 
previous state will be retained across redeployments, which can either be intentional or undesired depending on your use case.

````java
public class ASampleBean implements SmartBean {

  @Provided(entityId = "light.a_sample_light", supportBrightness = true, colorModes = Light.ColorMode.RGB)
  private ProvidedLight aLight;

  @Override
  public void init(SmartBeans sb) {
    aLight.setState(Light.State.ON);
    aLight.setBrightness(122);
    aLight.setRgbColor(new Light.RgbColor(0, 0, 255));
  }
}
````

## Create Entity Programmatically

In addition to the annotation-based approach, you can programmatically create a light using the 
`provideLight()` method of the `SmartBeans` API. This approach is useful when the entity's attributes are 
generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the internal ID, which must be unique within the scope of the bean; 
the second is the entity ID used in Home Assistant; and the third is an optional builder for setting all other 
attributes.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {
  @Override
  public void init(SmartBeans sb) {
    ProvidedLight providedLight = sb.provideLight("my_light", "light.a_sample_light", def -> def
        .setFriendlyName("A sample light")
        .setIcon("mdi:home")
        .setSupportBrightness(true)
        .setColorModes(Light.ColorMode.COLOR_TEMP)
    );
    providedLight.onTurnOn(evt -> sb.log("sample light turned on"));
  }
}
````
