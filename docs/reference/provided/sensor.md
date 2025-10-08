---
sidebar_position: 20
description: "Provided entity to publish any state to Home Assistant"
title: Sensor
---

# Provided Sensor

A provided sensor can represent any `integer`, `double` or `String` state of a bean as an entity in Home Assistant,
making it available for use in UI components or automations. To create a provided sensor, declare a field of type `ProvidedSensor` 
in your bean and annotate it with `@Provided`. SmartBeans will then automatically create the sensor in Home Assistant, 
generate the corresponding Java object to control it, and inject this object into the annotated field. You 
can update the sensor's state using the `setState()` method, which comes in three versions for `integer`, `double` and `String`.

A provided sensor can represent any `int`, `double`, or `String` state of a bean as an entity in Home Assistant,
making it available for use in UI components or automations. To create a provided sensor, declare a field of type 
`ProvidedSensor` in your bean and annotate it with `@Provided`. SmartBeans will then automatically create the sensor in
Home Assistant, generate the corresponding Java object to control it, and inject this object into the annotated field.
You can update the sensor's state using the `setState()` method, which is available in three variants for `int`, 
`double`, and `String` types.


````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  @Provided(entityId = "sensor.a_sample_sensor")
  private ProvidedSensor providedSensor;

  public void updateSensor() {
    providedSensor.setState("foo");
  }
}
````

:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no provided entities can be 
created, since they cannot be associated with a device.
:::

## Entity Configuration

The `@Provided` annotation can be used to configure the provided sensor in detail. It supports the following attributes:

| Attribute           | Description                                                                                                                                         |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `entityId`          | **Required**. The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. |
| `id`                | The internal ID of the entity. This ID must be unique within the scope of the bean device. If not specified, the field name is used.                |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                        |
| `icon`              | The icon shown in Home Assistant for this entity. If no icon is specified, it is derived from the device class.                                     |
| `deviceClass`       | The [device class](https://www.home-assistant.io/integrations/sensor/#device-class) of the entity in Home Assistant.                                |
| `unitOfMeasurement` | The unit of measurement used in Home Assistant. If not specified, it is derived from the device class.                                              |
| `stateClass`        | The [state class](https://developers.home-assistant.io/docs/core/entity/sensor/#available-state-classes) of the entity in Home Assistant.           |
| `options`           | If set, the sensor can only take values from this array.                                                                                            |
| `supportBrightness` | _Not used for sensors._                                                                                                                             |
| `colorModes`        | _Not used for sensors._                                                                                                                             |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Setting State and Attributes

As mentioned above, the state of the sensor can be updated using the `setState()` method. In addition, you can 
attach custom attributes to the sensor. The `ProvidedSensor` interface offers several `setAttribute()` methods for
different data types, such as `Integer`, `Boolean`, or `String`.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class DiskSpaceBean implements SmartBean {

  @Provided(
      entityId = "sensor.free_disk_space",
      friendlyName = "Free Disk Space",
      icon = "mdi:harddisk",
      deviceClass = "data_size",
      unitOfMeasurement = "GB"
  )
  private ProvidedSensor freeDiskSpace;

  public void updateSensor() {
    long freeSpace = getFreeDiskSpace();
    freeDiskSpace.setState((double)freeSpace / 1024d / 1024d / 1024d);
    freeDiskSpace.setAttribute("total_size", getTotalDiskSize());
  }
}
````

Keep in mind that the entity is not updated automatically when it is created or when the bean is deployed. If you want
the sensor to update immediately upon creation, you need to override the bean's `init()` method or use the `readState()`
method, which is called when the persistent bean state is loaded.

## Create Entity Programmatically

In addition to the annotation-based approach, you can programmatically create a binary sensor using the 
`provideSensor()` method of the `SmartBeans` API. This approach is useful when the entity's attributes are 
generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the internal ID, which must be unique within the scope of the bean; 
the second is the entity ID used in Home Assistant; and the third is an optional builder for setting all other 
attributes.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  private ProvidedSensor freeDiskSpace;

  @Override
  public void init(SmartBeans sb) {
    freeDiskSpace = sb.provideSensor("my_sensor", "sensor.free_disk_space", def -> def
        .setFriendlyName("Free Disk Space")
        .setIcon("mdi:harddisk")
        .setDeviceClass("data_size")
        .setUnitOfMeasurement("GB")
    );
  }
}
````
