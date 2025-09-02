---
sidebar_position: 20
description: "Provided entity to publish any on/off state to Home Assistant"
title: Binary sensor
---

# Provided binary sensor

A provided binary sensor can represent any on/off state of a bean as an entity in Home Assistant, making it available 
for use in UI components or automations. To create a provided binary sensor, declare a field of type `ProvidedBinarySensor` 
in your bean and annotate it with `@Provided`. SmartBeans will then automatically create the binary sensor in Home 
Assistant, generate the corresponding Java object to control it, and inject this object into the annotated field. You 
can update the sensor's state using the `setState()` method.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  @Provided(entityId = "binary_sensor.a_sample_sensor")
  private ProvidedBinarySensor providedBinarySensor;

  private void updateBinarySensor() {
    providedBinarySensor.setState(BinarySensor.State.ON);
  }
}
````
:::note
The `@SmartBeanDef(beanDevice = @BeanDevice)` annotation is required on your bean to create the corresponding bean device 
in Home Assistant. Without this annotation, no bean device will be created, and therefore no provided entities can be 
created, since they cannot be associated with a device.
:::

## Entity configuration

The `@Provided` annotation can be used to configure the provided binary sensor in detail. It supports the following 
attributes:

| Attribute           | Description                                                                                                                                         |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `entityId`          | **Required**. The `entity_id` of the entity created in Home Assistant. If an entity with this ID already exists, Home Assistant appends `_2` to it. |
| `id`                | The internal ID of the entity. This ID must be unique within the scope of the bean device. If not specified, the field name is used.                |
| `friendlyName`      | The friendly name of the entity displayed in Home Assistant.                                                                                        |
| `icon`              | The icon shown in Home Assistant for this entity. If no icon is specified, it is derived from the device class.                                     |
| `deviceClass`       | The [device class](https://www.home-assistant.io/integrations/binary_sensor/#device-class) of the entity in Home Assistant.                         |
| `unitOfMeasurement` | _Not used for binary sensors._                                                                                                                      |
| `stateClass`        | _Not used for binary sensors._                                                                                                                      |
| `options`           | _Not used for binary sensors._                                                                                                                      |

The entity ID, friendly name, and icon are only initial values set when the entity is created. They can later be 
modified by the user through the Home Assistant interface.

## Setting state and attributes

As mentioned above, the state of the binary sensor can be updated using the `setState()` method. In addition, you can 
attach custom attributes to the sensor. The `ProvidedBinarySensor` interface offers several `setAttribute()` methods for
different data types, such as `Integer`, `Boolean`, or `String`.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class DiskSpaceWatcher implements SmartBean {

  private static final long THRESHOLD = 1024 * 1024 * 500; //500 MB
  
  @Provided(
      entityId = "binary_sensor.disk_space_low",
      friendlyName = "Disk Space",
      deviceClass = "problem"
  )
  private ProvidedBinarySensor diskSpaceSensor;

  private void updateDiskSpaceSensor() {
    long available = getAvailableDiskSpace();
    diskSpaceSensor.setState((available < THRESHOLD) ? State.ON : State.OFF);
    diskSpaceSensor.setAttribute("available",  available);
  }
}
````

Keep in mind that the entity is not updated automatically when it is created or when the bean is deployed. If you want
the sensor to update immediately upon creation, you need to override the bean's `init()` method or use the `readState()`
method, which is called when the persistent bean state is loaded.

## Create entity programmatically

In addition to the annotation-based approach, you can programmatically create a binary sensor using the 
`provideBinarySensor()` method of the `SmartBeans` API. This approach is useful when the entity's attributes are 
generated dynamically through business logic and cannot be determined at compile time. 

The method accepts three arguments: the first is the internal ID, which must be unique within the scope of the bean; 
the second is the entity ID used in Home Assistant; and the third is an optional builder for setting all other 
attributes.

````java
@SmartBeanDef(beanDevice = @BeanDevice)
public class ASampleBean implements SmartBean {

  private ProvidedBinarySensor diskSpaceSensor;

  @Override
  public void init(SmartBeans sb) {
    diskSpaceSensor = sb.provideBinarySensor("my_sensor", "binary_sensor.disk_space_low", def -> def
        .setFriendlyName("Disk Space")
        .setDeviceClass("problem")
    );
  }
}
````
