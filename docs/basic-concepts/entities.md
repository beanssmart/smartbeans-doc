---
sidebar_position: 20
---

# Working with entities

In SmartBeans, Home Assistant entities are implemented as Java objects. Each Home Assistant domain is represented by a 
corresponding Java interface that defines interactions with entities of that specific type. For example, the `light` 
domain is represented by the `Light` interface.

Every entity object in SmartBeans maintains a direct connection to its counterpart in Home Assistant, allowing you to 
query state information, retrieve attributes, and invoke actions or services on the entity.

To access a Home Assistant entity in your SmartBean, you simply need to declare a field of the appropriate interface 
type in your SmartBean class and annotate it with `@Entity`, specifying the entity ID to establish the connection. 
During deployment, SmartBeans automatically injects the correctly instantiated Java object into the annotated field.

For example, to interact with the `light.kitchen_ceiling` entity in Home Assistant, you can define a field in your 
SmartBean class as follows:

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light ceilingLight;
  
}
````

That's it, nothing more is needed.

## Calling services

To call a service on an entity, you can use one of the methods provided by the entity interface. For each entity domain,
the most common service calls are implemented as corresponding method calls. For example, the `Light` interface has
a method `turnOn()` that invokes the `light.turn_on` service in Home Assistant. Calling this method on an entity
object automatically adds the correct entity_id as the target parameter to the service call, ensuring the appropriate 
entity is addressed.

Each method accepts parameters that correspond to the underlying service call's options. The `turnOn()` method, for
instance, takes a list of attributes which define how the light is activated, including brightness, color, transition 
time, and other parameters. These attributes are represented as `LightAttr` objects, which can be instantiated using the 
factory methods provided by the `LightAttr` class.

````java
import static com.github.beanssmart.ha.entities.LightAttr.*;

public class KitchenMotionControl implements SmartBean {

  @Entity( "light.kitchen_ceiling")
  private Light ceilingLight;

  public void turnLightOn() {
    ceilingLight.turnOn(brightnessPercent(70), colorName("red"), transition(Duration.ofMillis(500)));
  }
}
````

Other service calls are much simpler and sometimes don't even require any parameters. For example, the `close()` method 
of the `Cover` interface, which invokes the `cover.close_cover` service, takes no parameters. If you want to learn more
about the available service call methods, please refer to the [entity reference](../reference/entities) of a specific 
entity domain.

### Generic service calls

To invoke a service that is not exposed as a method on the entity interface, you can utilize the `callService()` method 
provided by the `SmartBeans` API. This API is accessible through the `SmartBeans` interface. You can inject an instance of 
this type into your SmartBean by simply declaring it as a field. The `callService()` method accepts a Service object 
parameter, which encapsulates the service name and the parameters to be passed to Home Assistant. You can conveniently 
instantiate this object using the static factory method provided by the `Service` class, along with its fluent API for
specifying parameters as demonstrated below:

````java
public class KitchenMotionControl implements SmartBean {

  private SmartBeans sb;

  @Entity( "light.kitchen_ceiling")
  private Light ceilingLight;

  public void turnLightOn() {
    sb.callService(Service.name("light.turn_on")
        .onTarget(Target.entity(ceilingLight))
        .withData(atts -> atts
            .setAttribute("brightness_pct", 100)
            .setAttributeList("rgb_color", 255, 0, 0)
            .setAttribute("transition", 0.5)
        )
    );
  }
}
````

## Retrieving state information

The state of an entity can be accessed through the entity object via its getter methods. Each entity interface 
implements the `getStateAsString()` method, which returns the raw state string as provided by Home Assistant. Many 
entity interfaces also offer specialized accessor methods that provide the state in more type-appropriate formats, such 
as enums, integers, or floating-point values for more convenient programmatic usage.

Here are several approaches to determine whether a motion sensor is currently detecting movement:

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("binary_sensor.kitchen_motion")
  private BinarySensor motionSensor;

  public boolean isMotionDetectedGenericState() {
    return motionSensor.getStateAsString().equals("on");
  }

  public boolean isMotionDetectedEnumState() {
    return motionSensor.getState() == BinarySensor.State.ON;
  }
  
  public boolean isMotionDetectedConvenienceMethod() {
    return motionSensor.isOn();
  }
}
````

### Attributes

To get the state attributes of an entity, the entity objects provide getter methods for the most common attributes. 
For example, the `Light` interface provides the `getBrightness()` method, which returns the brightness of the light 
as an integer between `0` and `255`.

Every entity object provides a generic `getAttributes()` method that returns an `Attributes` object, which offers access
to _all_ attributes of the entity. This object exposes type-specific accessor methods for retrieving attribute values in 
their appropriate data types. For example, `asString()` returns a `String` value, while `asInteger()` returns an `Integer` 
value.

For types that have both primitive and wrapper counterparts (such as `int`/`Integer`, `boolean`/`Boolean`), the API 
provides two distinct accessor methods:
1. Methods returning wrapper classes (e.g., `asInteger()`) which return `null` when the attribute is not set
2. Methods returning primitive types (e.g., `asInt()`) which accept a default value parameter that is returned when 
   the attribute is not set

This dual approach allows for both null-safety with wrapper types and default value handling with primitive types.

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light ceilingLight;

  public void checkIfLightIsBright() {
    if(ceilingLight.getBrightness() > 200) {
      
    }
    if(ceilingLight.getAttributes().asInt("brightness", 0) > 200) {
      
    }
  }
}
````

This generic approach also enables access to attribute lists and handles traversal of nested attributes within object 
hierarchies. While you will primarily encounter this pattern in specific edge cases, understanding its capabilities is 
valuable. Consider the following examples:

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("light.kitchen_ceiling")
  private Light ceilingLight;

  public void accessAttributes() {
    //List of Integer
    List<Integer> colorValues = ceilingLight.getAttributes().asList("rgbColor").stream()
        .map(Attribute::asInteger)
        .toList();
    
    //Nested objects
    Attributes nested = ceilingLight.getAttributes().asObject("nested");
    nested.asString("attr_1");
    
    //Or even a list of nested objects
    List<Attributes> nestedObjects = ceilingLight.getAttributes().asList("nested_list").stream()
            .map(Attribute::asObject)
            .toList();
  }
}
````

## State listeners

You can attach listeners to entity objects to receive notifications whenever the state of the entity or any of its 
attributes changes. To register a listener, use the `onStateChanged()` method available on every entity. You must 
provide an `EntityStateListener` implementation that will be notified of state or attribute changes. 

The listener interface defines a single method, `stateChanged()`, which is invoked upon any state or attribute 
modification. This method receives an `EntityStateChangedEvent<Entity>` parameter containing a reference to the modified
entity object.

````java
public class KitchenMotionControl implements SmartBean {

  @Entity("binary_sensor.motion_kitchen")
  private BinarySensor motionSensor;

  public void registerMotionListener() {
    motionSensor.onStateChanged(event -> {
      if(event.getEntity().isOn()) {
        //motion detected
      }
    });
  }
}
````
:::note
[In the following section](trigger), we'll explore a more sophisticated approach to responding to state changes: directly
registering for a [trigger](trigger) that executes when specific state conditions are met. This method typically offers
greater precision, as it allows you to respond to well-defined state transitions rather than relying on generic change
events. With state triggers, you can define exact conditions including source and target states, attribute-specific
changes, and even duration requirements.
:::
