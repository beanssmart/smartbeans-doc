---
sidebar_position: 60
---

# Bean Devices

:::note
Bean devices require the MQTT integration in Home Assistant. They are only available when you have an MQTT broker
running and properly configured in Home Assistant. The devices and entities described in this and the following chapters
appear as MQTT entities in Home Assistant and are automatically created through the MQTT integration's discovery mechanism. 
Therefore, [MQTT discovery](https://www.home-assistant.io/integrations/mqtt/#mqtt-discovery) must be enabled in your 
installation.
:::

Each bean instance in SmartBeans can create a device in Home Assistant. This device acts as a container that bundles all
entities created by the bean. The following chapters describe the types of entities that can be created by a bean 
instance and their configuration options.

The primary entity is the "bean entity" which provides information about the bean instance and optionally allows 
enabling or disabling the entire bean. When a bean is disabled, the framework will not invoke any bean methods. For 
example, if you have implemented a bean for motion-controlled lighting that uses several triggers to detect motion,
disabling it will deactivate the entire motion-control functionality.

To define the "bean entity" use the `beanDevice` attribute in the `@SmartBeanDef` annotation and specify the desired 
entity type. Available options are `SWITCH` and `BINARY_SENSOR`. A `SWITCH` entity allows toggling the bean between 
enabled and disabled states. A `BINARY_SENSOR` entity provides the same information about the bean's status but does not 
offer control functionality to enable or disable the bean.

````java
@SmartBeanDef(beanDevice = @BeanDevice(type = BeanDevice.EntityType.SWITCH))
public class KitchenMotionControl implements SmartBean {

}
````

This results in a device and switch entity in Home Assistant that looks like this:

![Bean device in Home Assistant](/img/screenshots/device_simple.png)

The attributes of the bean entity provide some useful metadata about the bean instance, including bean name,
implementing class or the values of bean parameters. Consider this example:

````java
@SmartBeanDef(
    name = "KitchenMotion",
    beanDevice = @BeanDevice(type = BeanDevice.EntityType.SWITCH),
    params = {
        @Param(name = "light", value = "light.kitchen_ceiling"),
        @Param(name = "motion", value = "binary_sensor.kitchen_motion")
    }
)
public class MotionControl implements SmartBean {

}
````

The bean entity in Home Assistant will have the following attributes:

![Bean device in Home Assistant](/img/screenshots/bean_entity_attributes.png)

